#### 前言

最近在工作中遇到了一个需求，会用到`EventBus`，正好看到`eShopOnContainers`上有相关的实例，去研究了研究。下面来分享一下用`EventBus` 来改造一下我们上篇[Event发布与实践](http://www.cnblogs.com/xiyin/p/8806440.html) 中所用的`Event`。

在[上一篇](http://www.cnblogs.com/xiyin/p/8806440.html)中讲到` Event `在发布与订阅模式中的一些实例，接下来实践一下通过把上面的例子改造成`EventBus`来加深理解。也感谢参考资料中大佬前辈们的思想和精华。

#### 理解事件及其本质

我们所构建的一个场景是这样的：有个`CarManager`类，其中有个发车了的事件名为`CarNotification`， 有司机`Driver`和乘客`Passenger`这两个类。分别订阅了 发车事件， 这里司机和乘客收到通知后，然后简单的处理，仅仅打印出 司机和乘客的姓名信息。

在上面的场景中，我们可以知道，事件源，事件处理各是什么。

事件源：司机 或者 乘客 类。

事件处理： 打印出 司机或 乘客的信息。

大概花了一张巨丑的图，下面：

> ![](https://ws4.sinaimg.cn/large/006tKfTcgy1fqi4ws4pq0j31240sa420.jpg)





#### 开始抽象事件源

接下来，我们可以开始抽象起来了。首先是 事件源：

定义一个所有事件源的父类，名为 `EventData`: 所有的事件源都需要继承该类

```c#
public class EventData{
    public Guid Id{ get; }
    public DateTime CreationDate{ get; }
    
    public EventData(){
        Id = Guid.NewGuid();
        CreationDate = DateTime.Now;
    }
}
```

然后我们就可以把我们之前的`Driver` 和`Passenger` 用我们定义的事件源来改造一下：

```c#
public class CarNotificationEventData : EventData{
    private string _driverName;
    private string _passengerName;
   	
    public CarNotification(string driverName, string passengerName){
        _driverName = driverName;
        _passengerName = passengerName;
    }
    
    public string Driver{ get{ return _driverName; } }
    
    public string Passenger{ get{return _passengerName; } }
}
```

接下来就是抽象 事件处理 了,我们在此定义一个名为 IEventDataHandler 的接口:

```c#
public interface IEventHandler<in TEventData> : IEventHandler where TEventData : EventData{
    Task Handle(TEventData eventData);
}

public interface IEventHandler{
    
}
```

* 定义了一个泛型接口，参数必须继承自 EventData 类型。
* 一个方法 Handle 方法，接收的参数也为 EventData 类型。

查看`eShopOnContainer`的源码时，上面那个为啥定义，继承一个空接口`IEventHandler` ，当时有点没搞明白，后来继续去看了看源码，发现了下面这段：

```c#
var concreteType = typeof(IIntegrationEventHandler<>).MakeGenericType(eventType);
```

**大胆猜测一下，应该是反射时会用到，所以定义了一个空接口**

那么我们根据上面的接口，简单改造一下我们的事件处理代码：

```c#
public class DriverHandler : IEventHandler<CarNotificationEventData>{
    public void Handle (CarNotificationEventData carNotificationEventData) {
            Console.WriteLine ("Driver Hanlder---------");
            Console.WriteLine (carNotificationEventData.Driver + "\n" + carNotificationEventData.Passenger +
                "\n" + carNotificationEventData.EventDate);
        }
}
```

偷了个懒，简单了打印了以下事件源中的信息，当然生产环境中，你就根据自己的业务逻辑来进行处理。



当然，改了事件源和事件处理，当然也需要重新对 我们当初 事件 以及 委托的定义。

```c#
//修改委托的定义：

public delegate void CarEventDataHandler(CarNotificationEventData eventData);

//修改事件的定义：
public event CarEventDataHandler CarNotification;
```



***

实现到上面那步，其实我们还只是刚刚开始，因为你会发现，我们只仅仅把那些事件源和事件处理抽线了出来，在每个事件处理程序中，我们可能还需要通过

但并没有真正上的做到用事件总线来实现。

为了更好的实现下面的事件总线，我们把委托也单独定义到一个class 中：

```c#
namespace EventDemo{
    public delegate void CarNotificationDelegate(CatNotificationEventData eventData);
}
```



#### 实现事件总线

根据 eShopOnContainers上的源码 [IEventBusSubscription](https://github.com/doublnt/eShopOnContainers/blob/dev/src/BuildingBlocks/EventBus/EventBus/IEventBusSubscriptionsManager.cs)，我们来实现一个基于InMemory（存在Dictionary 中）的事件总线。

首先想到的 发布与订阅模式，所以呢，这个事件总线里面一定要有可以订阅和移除订阅的方法，还有来个额外的判断当前事件总线是否为空，当然还有一个就是 事件，接下来我们就定义一个 IEventBusSubscription：

```c#
 public interface IEventBusSubscriptionsManager {
        bool IsEmpty { get; }

        event CarNotificationDelegate OnEventRemoved;

        void AddSubscription<T, TH> () where T : CarNotificationEventData where TH : IEventHandler<T>;

        void RemoveSubscription<T, TH> (T eventBusData) where T : CarNotificationEventData where TH : IEventHandler<T>;

        bool HasSubscriptionsForEvent<T> () where T : CarNotificationEventData;

        bool HasSubscriptionsForEvent (string eventName);

    }
```

就简单点，一个订阅方法，一个移除订阅方法。还有一个事件 OnEventRemoved

接下来就是实现了，eShopOnContainer 上面有好多个版本，我在实践中，试了试 InMemory 和 RabbitMQ 的。因为十分贴合我的业务场景，本篇先介绍一下InMemory的实现，因为对RabbitMQ理解的还不是很深入， RabbitMQ版本的后续博文中跟进。

然后我们建立一个InMemoryEventBusSubscriptionsManager 继承至IEventBusSubscripionsManager

```c#
public class InMemoryEventBusSubscriptionsManager : IEventBusSubscriptionsManager {
        private readonly Dictionary<string, List<Type>> _handler; //Type is HandlerType
        private readonly List<Type> _eventTypes;
        public event CarNotificationDelegate OnEventRemoved;
        private readonly IServiceProvider _service;

        public InMemoryEventBusSubscriptionsManager (IServiceCollection service) {
            _handler = new Dictionary<string, List<Type>> ();
            _eventTypes = new List<Type> ();
            _service = service.BuildServiceProvider ();
            OnEventRemoved += BeiginProcess;
        }

        public bool IsEmpty => !_handler.Keys.Any ();

        public void AddSubscription<T, TH> ()
        where T : CarNotificationEventData
        where TH : IEventHandler<T> {
            var eventName = GetEventKey<T> ();

            if (!HasSubscriptionsForEvent<T> ()) {
                _handler.Add (eventName, new List<Type> ());
            }

            if (_handler[eventName].Any (t => t == typeof (TH))) {
                throw new ArgumentException (
                    $"Handler Type {typeof(TH).Name} already registered");
            }
            _handler[eventName].Add (typeof (TH));

            _eventTypes.Add (typeof (T));
        }

        public void RemoveSubscription<T, TH> (T eventData)
        where T : CarNotificationEventData
        where TH : IEventHandler<T> {
            var handlerToRemove = FindSubscriptionToRemove<T, TH> ();
            DoRemoveHandler (eventData, handlerToRemove);
        }

        private void DoRemoveHandler (CarNotificationEventData eventData, Type subsToRemove) {
            if (subsToRemove != null) {

                var eventName = eventData.GetType ().Name;

                _handler[eventName].Remove (subsToRemove);
                if (!_handler[eventName].Any ()) {
                    _handler.Remove (eventName);
                    var eventType = _eventTypes.SingleOrDefault (e => e == eventName.GetType ());
                    if (eventType != null) {
                        _eventTypes.Remove (eventType);
                    }
                    RaiseOnEventRemoved (eventData);
                }

            }
        }

        private void RaiseOnEventRemoved (CarNotificationEventData eventData) {
            var handler = OnEventRemoved;
            if (handler != null) {
                OnEventRemoved (eventData);
            }
        }

        private Type FindSubscriptionToRemove<T, TH> ()
        where T : CarNotificationEventData
        where TH : IEventHandler<T> {
            var eventName = GetEventKey<T> ();
            return DoFindSubscriptionToRemove (eventName, typeof (TH));
        }

        private Type DoFindSubscriptionToRemove (string eventName, Type handlerType) {
            if (!HasSubscriptionsForEvent (eventName)) {
                return null;
            }

            return _handler[eventName].SingleOrDefault (s => s == handlerType);

        }
        public bool HasSubscriptionsForEvent<T> () where T : CarNotificationEventData {
            var keyName = GetEventKey<T> ();
            return _handler.ContainsKey (keyName);
        }

        public bool HasSubscriptionsForEvent (string eventName) => _handler.ContainsKey (eventName);

        public string GetEventKey<T> () {
            return typeof (T).Name;
        }

        public Type GetEventTypeByName (string eventName) => _eventTypes.SingleOrDefault (t => t.Name == eventName);

        public async void BeiginProcess (CarNotificationEventData eventData) {
            await Process (eventData);
        }

        private async Task Process (CarNotificationEventData eventBusData) {
            var eventName = eventBusData.GetType ().Name;
            if (HasSubscriptionsForEvent (eventName)) {
                var subscriptions = _handler[eventName];

                foreach (var subscription in subscriptions) {
                    var eventType = GetEventTypeByName (eventName);
                    var handler = _service.GetService (subscription);
                    var concreteType = typeof (IEventHandler<>).MakeGenericType (eventType);

                    await (Task) concreteType.GetMethod ("EventHandle").Invoke (handler, new object[] { eventBusData });
                }
            }
        }
    }
```

我稍微改动了以下地方的代码，使我的实例更加符合场景上的运行，

* eShopOnContainer 里面使用了 Autofac 来运用DI，我直接使用了 .net -core 中自带的 IServiceProvider来替代，感觉更加方便
* 直接对外显示一个public 的BeginProcess 来引发事件，主要为了演示方便。

既然EventBus 都写好，我们可以开始运行了，

```c#
//... 
public static void Main (string[] args) {
            #region EventBusRegister Demo

            var serviceProvider = new ServiceCollection ()
                .AddSingleton<IEventBusSubscriptionsManager, InMemoryEventBusSubscriptionsManager> ()
                .AddTransient<DriverHandler> ();

            var eventBus = new InMemoryEventBusSubscriptionsManager (serviceProvider);

            RegisterEventBus (eventBus);

            CarNotificationEventData carNotificationEventData = new CarNotificationEventData ("Robert 1", "Passenger 1");

            eventBus.BeiginProcess (carNotificationEventData);
            #endregion
        }
//...
```

dotnet run  运行一下，得到如下结果：

> ![](https://ws4.sinaimg.cn/large/006tNc79gy1fqlj14v48oj30zy04q3yx.jpg)



这样就算大功告成了。接下来会写一篇结合RabbitMQ 的EventBus ，就更加符合生产环境的情景了。文中如果解释的不到位处，欢迎评论中指出，一起探讨。



#### 参考资料

* [事件总线知多少（1）](http://www.cnblogs.com/sheng-jie/p/6970091.html)
* [DDD事件总线的实现](http://www.cnblogs.com/dehai/p/4887998.html)