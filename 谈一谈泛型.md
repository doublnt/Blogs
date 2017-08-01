谈一谈泛型

首先，泛型是`C#2`出现的。这也是`C#2 `一个重要的新特性。泛型的好处之一就是在编译时执行更多的检查。

**泛型类型和类型参数**

​    泛型的两种形式：泛型类型（ 包括类、接口、委托和结构  **没有泛型枚举**）

下面以`Dictionary<TKey,TValue>` 来为例，解释一下`类型参数`，`类型实参（type argument）`，`开放类型`，`封闭类型`

上面的例子中，

* 类型参数是 `TKey` 和`TValue`。
* 在使用泛型类型或方法时，要用真实的类型代替，比如 `Dictionary<string,int>`,那么类型实参就是 `string` 和`int`。
* 如果没有为泛型参数提供类型实参，那么这就是一个`未绑定泛型类型(unbound generic type)`，如果指定了实参，该类型就称为一个`已构造类型(constructed type)`
* 已构造类型可以是开放类型（open type）或封闭类型（closed type），开放类型 还包含一个类型参数（例如这样`Dictionary<T,int>`）,而封闭类型就像这样 `Dictionary<string,int>`。

> PS：我觉得吧，记住一些专业术语还是很重要的。比如泛型（`Generic`），实参（argument）和形参（parameter）以及上面的一些专业术语，因为当你在使用 Google or Stackoverflow时，别到时知道问题但是你输入不来关键字。还有 `List<T> `读作 `List of T`

接下来谈谈类型约束（`Type Constraint`）与类型推断（`Type Inference`）。

首先是 类型约束，主要有四种约束（`Constraint`）可供使用，约束要放到泛型方法或泛型类型声明的末尾，并由上下文关键字`Where` 来引入。

主要有以下四种约束分别是，引用类型约束、值类型约束、构造函数类型约束、转换类型约束。

* **引用类型约束**主要确保使用的类型实参是引用类型 由 `T:Class` 表示，**并且必须是类型参数指定的第一个约束**。

  > `struct RefSample<T> where T : class`

  比如 `RefSample<string>` 其类型实参是引用类型，但是 `RefSample<string>` 仍然是值类型。

* 值类型约束示例声明如下

  > `class ValSample<T> where T : struct`

* 构造函数类型约束表示为 ` T: new()`，并且必须是所有类型参数的**最后一个约束**。

* 转换类型约束允许你指定另一个类型，类型实参必须可以通过一致性、引用、或者装箱隐式地转换为该类型。这里就引出了另一个名词 类型参数约束（`type parameter constraint`）

* 最后约束一般都是组合起来使用的，下面就是组合约束时需要注意的一些事项。

  * 不能既是引用类型又是值类型
  * 如果已经有一个值类型约束，就不允许再指定一个构造函数约束（但如果，`T` 被约束成一个值类型，仍可以方法内部使用 `new T()`）
  * 如果存在多个转换类型约束，并且其中一个为类，那么它应该出现在接口的前面，而且我们不能多吃指定同一个接口。

  > 接下来展示几个有效的和无效的例子
  >
  > ```c#
  > class Sample<T> where T : class, IDisposable, new ()   is Valid
  > class Sample<T,U> where T : Stream where U : IDisposable  is valid
  >   
  > class Sample<T> where T : class, struct is invalid
  > class Sample<T,U> where T : Stream, U : IDisposable is invalid
  > ```

> **泛型比较接口** ： `IComparer<T>`  和 `IComparable<T>` 用于排序（判断猴哥值是小于、等于、还是大于另一个值），而`IEqualityComparer<T>` 和 `IEquatable<T>` 通过某种标准来比较两个项的相等性，或查找某个列的散列。

可变性 分为 协变性和逆变性，在C#4.0 之前，泛型是不支持可变性的，之后的话，就出现了 `out` 和 `in`。