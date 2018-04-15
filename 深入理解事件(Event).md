### 前言

在前一篇文章中讲到了[Event 发布与订阅(一)](http://www.cnblogs.com/xiyin/p/8806440.html) 里面用到了事件来实现一些发布与订阅，当时对事件及其委托理解的还不是太深入，可能在使用上有点捉急。这篇来好好讲讲事件，以及通过一些小DEMO来加深理解。可以说是让我重新理解了事件。

### 事件(Event)

事件： 

> 定义了事件成员的类型允许类型（或类型的实例）通知其他对象发生特定的事情。例如：Button类提供了Click事件。应用程序中的一个或多个对象可接受关于该事件的通知，以便在Button被单击之后采取特定操作。

定义了事件成员的类型能提供以下功能。

- 方法能登记它对事件的关注
- 方法能注销它对事件的关注
- 事件发生时，登记了的方法将收到通知

CLR事件模型以**委托**为基础。委托时调用回调方法的一种类型安全的方式。对象凭借回调方法接受它们订阅的通知。

### 核心指南

想要实现一个事件，按照下面这四个步骤走：

#### 1. 定义附加信息

**定义类型来容纳所有需要发送给事件通知接收者的附加信息**

这句话怎么理解呢，就以之前我们那个Car为例子。把它抽象出事件来看，我要定义一个开车通知事件，我想要知道的是 司机 和乘客都是谁：那么我们就定义一个长这样的:

```c#
using System;
using EventDemo.EventBus;

namespace EventDemo.CarDemo {
    public class CarNotificationEventData : EventData {
        private string _driverName;
        private string _passengerName;

        public CarNotificationEventData (string driver, string passenger) {
            _driverName = driver;
            _passengerName = passenger;
        }

        public string Driver { get { return _driverName; } }

        public string Passenger { get { return _passengerName; } }

        public DateTime NotifiDate { get { return DateTime.Now; } }
    }
}
```

那么我们就完成第一步了。

#### 2. 定义事件成员

```c#
using System;
using System.Threading;

namespace EventDemo.CarDemo {
    //...

    public class CarManager {

        //定义一个 上车通知 事件 成员类型为 CarNoticationEventHandler
        public event CarNotificationEventHandler CarNotification;

        //...
}
```

CarNotification 是事件名称，事件成员的类型为 CarNotificationEventHandler。 这意味着 事件通知的 所有接收者都必须提供一个原型和 CarNotificationHandler 委托类匹配的回调方法。

CarNotificationHandler 的定义为：

```c#
    public delegate void CarNotificationEventHandler (CarNotificationEventData eventData);
```

#### 3. 定义负责引发事件的方法来通知事件的登记对象

我在CarManager 中当然是定义一个 开车了 方法，来通知所有该事件的接受者说明车要开了。

```c#
 public void OnCarToRun (CarNotificationEventData carNotificationEventData) {

            //将委托字段的引用复制到一个临时变量中，出于线程安全的考虑
            CarNotificationEventHandler temp = Volatile.Read (ref CarNotification);

            if (temp != null) {
                temp (carNotificationEventData);
            }
        }
```

#### 4. 定义方法将输入转化为期望事件

这就是步骤中的最后一步了，就是引发这个事件。在我们的场景中就是，把司机和乘客都叫上车，然后我们就发车了。

```c#
public void RunCar (Driver driver, Passenger passenger) {
            CarNotificationEventData carNotificationEventData = new CarNotificationEventData (driver.Name, passenger.Name);
            OnCarToRun (carNotificationEventData);
        }
```

把司机和乘客，添加到附加信息对象里面。然后就引发这个通知事件。



***



通过上面那四个步骤，应该对事件有个比较深入的了解,最后只需在各个事件的接受者里定义回调方法的处理就可以啦。

这里给出其中一个处理：

```c#
 public void DriverHandle (CarNotificationEventData carNotificationEventData) {
            Console.WriteLine ("Driver Hanlder---------");
            Console.WriteLine (carNotificationEventData.Driver + "\n" + carNotificationEventData.Passenger +
                "\n" + carNotificationEventData.NotifiDate);
        }
```

至此，整个事件的发布，接收和处理都已完毕。如有错误的地方，一起来讨论讨论。