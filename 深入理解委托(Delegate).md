### 委托（Delegate）

说起委托，我们首先应该想到 **回调函数** .NET-Framework  通过**委托** 来提供回调函数机制。委托确保回调方法是类型安全的，委托还允许调用多个方法，并支持静态方法和实例方法。

下面通过一个具体实例来分析委托的声明，创建，以及使用。

```c#
public sealed class Program {
   public static void Main() {
      DelegateIntro.Go();
      GetInvocationList.Go();
      AnonymousMethods.Go();
      DelegateReflection.Go("TwoInt32s", "Add", "123", "321");
      DelegateReflection.Go("TwoInt32s", "Subtract", "123", "321");
      DelegateReflection.Go("OneString", "NumChars", "Hello there");
      DelegateReflection.Go("OneString", "Reverse", "Hello there");
   }
}


internal sealed class DelegateIntro {
   // Declare a delegate type; instances refer to a method that
   // takes an Int32 parameter and returns void.
   internal delegate void Feedback(Int32 value);

   public static void Go() {
      StaticDelegateDemo();
      InstanceDelegateDemo();
      ChainDelegateDemo1(new DelegateIntro());
      ChainDelegateDemo2(new DelegateIntro());
   }

   private static void StaticDelegateDemo() {
      Console.WriteLine("----- Static Delegate Demo -----");
      Counter(1, 3, null);
      Counter(1, 3, new Feedback(DelegateIntro.FeedbackToConsole));
      Counter(1, 3, new Feedback(FeedbackToMsgBox)); // "Program." is optional
      Console.WriteLine();
   }

   private static void InstanceDelegateDemo() {
      Console.WriteLine("----- Instance Delegate Demo -----");
      DelegateIntro di = new DelegateIntro();
      Counter(1, 3, new Feedback(di.FeedbackToFile));

      Console.WriteLine();
   }

   private static void ChainDelegateDemo1(DelegateIntro di) {
      Console.WriteLine("----- Chain Delegate Demo 1 -----");
      Feedback fb1 = new Feedback(FeedbackToConsole);
      Feedback fb2 = new Feedback(FeedbackToMsgBox);
      Feedback fb3 = new Feedback(di.FeedbackToFile);

      Feedback fbChain = null;
      fbChain = (Feedback)Delegate.Combine(fbChain, fb1);
      fbChain = (Feedback)Delegate.Combine(fbChain, fb2);
      fbChain = (Feedback)Delegate.Combine(fbChain, fb3);
      Counter(1, 2, fbChain);

      Console.WriteLine();
      fbChain = (Feedback)Delegate.Remove(fbChain, new Feedback(FeedbackToMsgBox));
      Counter(1, 2, fbChain);
   }

   private static void ChainDelegateDemo2(DelegateIntro di) {
      Console.WriteLine("----- Chain Delegate Demo 2 -----");
      Feedback fb1 = new Feedback(FeedbackToConsole);
      Feedback fb2 = new Feedback(FeedbackToMsgBox);
      Feedback fb3 = new Feedback(di.FeedbackToFile);

      Feedback fbChain = null;
      fbChain += fb1;
      fbChain += fb2;
      fbChain += fb3;
      Counter(1, 2, fbChain);

      Console.WriteLine();
      fbChain -= new Feedback(FeedbackToMsgBox);
      Counter(1, 2, fbChain);
   }

   private static void Counter(Int32 from, Int32 to, Feedback fb) {
      for (Int32 val = from; val <= to; val++) {
         // If any callbacks are specified, call them
         if (fb != null)
            fb(val);
      }
   }

   private static void FeedbackToConsole(Int32 value) {
      Console.WriteLine("Item=" + value);
   }

   private static void FeedbackToMsgBox(Int32 value) {
      MessageBox.Show("Item=" + value);
   }

   private void FeedbackToFile(Int32 value) {
      StreamWriter sw = new StreamWriter("Status", true);
      sw.WriteLine("Item=" + value);
      sw.Close();
   }
}

```

在上面，Feedback 委托指定的方法接受Int32参数，返回 void。

将方法绑定到委托时，C#和CLR都允许引用类型的协变性（covariance）和逆变性(contravariance)。协变性是指方法能返回从委托的返回类型派生的一个类型。逆变性是指方法获取的参数可以是委托的参数类型的基类。关于可变性和逆变性可以参考一下之前的博文[接口和委托的泛型可变性](http://www.cnblogs.com/xiyin/p/7295991.html) 

假设定义如下委托

`delegate Object MyCallback(FilStram s);`

完全可以构造如下的方法：

`String SomeMethod(Stream s);`

SomeMethod 的返回类型（String） 派生自委托的返回类型（Object）这种协变性是允许的。SomeMethod 的参数类型（Stream）是委托的参数类型（FileStream）的基类。这种逆变性是允许的。

**只有引用类型才支持协变性和逆变性，值类型或void 不支持**

#### 委托揭秘

首先让我们再来看一下这个委托的声明代码：

`internal delegate void Feedback(Int32 value);`

编译器会做这样一件事，把上面那段代码定义成下面这一段

```c#
internal class Feedback :System.MulticastDelegate{
    //构造器
    public Feedback(Object @object, Intptr method);
    
    public virtual void Invoke(Int32 value);
    
    public virtual IAsyncResult BeginInvoke(Int32 value, AsyncCallback callback, Object @object);
    
    public virtual void EndInvoke(IAsyncResult result);
}
```



Feedback 类派生自FCL（Framework class library）定义的 System.MulticastDelegate 类

> 所有委托类型都派生自MulticastDelegate

> **委托类既可以嵌套在一个类型中定义，也可以在全局范围中定义。简单来说，由于委托是类，所以凡是能够定义类的地方，都能定义委托。**



接下来介绍一下MulticastDelegate 中三个重要的非公共字段

>1. _target  类型为 System.Object ，当委托队形包装一个静态方法时，这个字段为null。当委托对象包装一个实例方法时，这个字段引用的是回调方法要操作的对象。换言之，这个字段指出要传给实例方法的隐式参数this 的值。
>2. _methodPtr 类型为 System.IntPtr 一个内部的整数值,CLR 用它来标识要回调的方法
>3. _invocationList 类型为 System.Object 该字段通常为 null。构造委托链时它引用一个委托数组



比如上面代码中的

`Feedback fb1 = new Feedback(FeedbackToConsole)`



那么这个 fb1 的状态就如下面这张图这样：

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fqc8ibamr0j30iw0acq3h.jpg)



下面贴上原书中委托链的状态图（偷个懒，太难画了），

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fqc8jue1z7j31kw23v7wi.jpg)



下面给出一个我本地委托测试的小Demo



Driver.cs

```c#
using System;

namespace EventDemo.CarDemo {
    public class Driver {
        public string Name { get; set; }

        public void DriveCar () {
            Console.WriteLine ($"I am driver:{Name}");
        }
    }
}
```

Passenger.cs

```c#
namespace EventDemo.CarDemo {
    public class Passenger {

        public string Name { get; set; }

        public void BoardCar () {
            System.Console.WriteLine ($"我上车了，我是：{Name}");
        }
    }
}
```

MyCar.cs

```c#
using System;

namespace EventDemo.CarDemo {

    public delegate void CarHandler ();

    public class MyCar {
        //定义一个 上车 委托方法的事件
        public event CarHandler CarNumberNotification;

        public void RunCar () {
            Console.WriteLine ($"好的，准备上车了！");
            if (CarNumberNotification != null) {
                CarNumberNotification ();
            }
        }
    }
}
```

Program.cs

```c#
 //纯委托版本
CarHandler carHandler = null;
carHandler += driver.DriveCar;
carHandler += passenger.BoardCar;
carHandler.Invoke();
```



