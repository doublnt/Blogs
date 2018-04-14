#### 前言

主要讲的是发布与订阅在Event中的一个简单实现用来加深理解。

#### C #中的事件(Event)的理解：

事件具有以下属性：(From [Events](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/events/))

- 发行者确定何时引发事件；订户确定对事件作出何种响应。
- 一个事件可以有多个订户。 订户可以处理来自多个发行者的多个事件。
- 没有订户的事件永远也不会引发。
- 事件通常用于表示用户操作，例如单击按钮或图形用户界面中的菜单选项。
- 当事件具有多个订户时，引发该事件时会同步调用事件处理程序。 若要异步调用事件，请参阅 [Calling Synchronous Methods Asynchronously](https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/calling-synchronous-methods-asynchronously)。
- 在 .NET Framework 类库中，事件基于 [EventHandler](https://docs.microsoft.com/dotnet/api/system.eventhandler) 委托和 [EventArgs](https://docs.microsoft.com/dotnet/api/system.eventargs) 基类。

#### 发布与订阅(Observer, Publish-Subscribe)

意图：

&nbsp;定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并主动更新。解耦

别名： 依赖(Dependents)，发布-订阅(Publish-Subscribe)

&nbsp;Observer模式描述了如何建立这种关系。这一模式中的关键对象是目标(Object)和观察者(observer)。一个目标可以有任意数目的依赖它的观察者。一旦目标的状态发生改变，所有的观察者都得到通知。作为对这个通知的响应，每个观察者都将查询目标以使其状态与目标的状态同步。

&nbsp;这种交互也称为 发布-订阅。目标是通知的发布者，它发出通知时并不需要知道谁是它的观察者。可以有任意数目的观察者订阅并接收通知。

#### 发布与订阅的实践

假设一个这样的场景。车类，以及乘客类

车类里：有一个事件，开车通知。有一个方法，开车。

调用开车方法时，通知订阅了开车通知的观察者，然后开车。

首先是定义一个车类：

```c#
using System;

namespace EventDemo {
    public class MyCar {
        //定义一个 上车 的委托
        public delegate void BeginOnCarHandler ();

        //定义一个 上车 委托方法的事件
        public event BeginOnCarHandler CarNotification;

        public string Name { get; set; }

        public void RunCar () {
            CarNotification ();
            Console.WriteLine ("好的，都上车了司机" + Name + "，开车了");
        }
    }
}
```

然后定义一个乘客类：

```c#
namespace EventDemo {
    public class Passenger {

        public string Name { get; set; }

        public void BeginToCar () {
            System.Console.WriteLine ("我上车了，我是：" + Name);
        }
    }
}
```

最后执行：

```c#
MyCar benz = new MyCar { Name = "Benz" };

            Passenger p4 = new Passenger { Name = "xiaoming" };
            Passenger p2 = new Passenger { Name = "xiaohong" };

            benz.CarNotification += new MyCar.BeginOnCarHandler(p2.BeginToCar);
            benz.CarNotification += new MyCar.BeginOnCarHandler(p4.BeginToCar);

            benz.RunCar ();
```

最后的结果为 

> ![](https://ws2.sinaimg.cn/large/006tNc79gy1fq9pb3duwdj315y03cq3x.jpg)

上面的方法

```c#
benz.CarNotification += new MyCar.BeginOnCarHandler(p2.BeginToCar);
```

也可以改成

```c#
benz.CarNotification += p2.BeginToCar;
```



上面就是一个简单的发布/订阅的例子，在EventBus中也有其涉及的地方，正好可以了解记录一下。

今天下午又仔细思考了一下，更新了一下实例，如下：

一个车，是被观察者，也就是目标 object，它有一个事件，暂且叫它 发车事件。以及观察者 乘客类。

修改了一下Event代码:

```c#
using System;

namespace EventDemo {
    public class MyCar {
        //定义一个 上车 的委托
        public delegate void CarHandler (MyCar car);

        //定义一个 上车 委托方法的事件
        public event CarHandler CarNumberNotification;

        public string Name { get; set; }

        public int Count { get; set; }

        public void RunCar () {
            Console.WriteLine ($"好的，准备上车了，车名为:{Name}，车上人数:{Count}");
            if (CarNumberNotification != null) {
                CarNumberNotification (this);
            }
            Console.WriteLine ($"The people now is:{Count}");
        }
    }
}
```

```c#
namespace EventDemo {
    public class Passenger {

        public string Name { get; set; }

        public void BeginToCar (MyCar car) {
            car.Count++;
            System.Console.WriteLine ($"我上车了，我是：{Name}，我坐的车是{car.Name}");
        }
    }
}
```



```c#
            MyCar benz = new MyCar { Name = "Benz" };

            Passenger p1 = new Passenger { Name = "xiaoming" };
            Passenger p2 = new Passenger { Name = "xiaohong" };

            benz.CarNumberNotification += p1.BeginToCar;
            benz.CarNumberNotification += p2.BeginToCar;

            benz.RunCar ();
```

这样编写代码更加容易理解一些。