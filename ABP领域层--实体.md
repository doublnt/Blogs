[TOC]

**标题：重温ABP领域层**



## 1. 前言

 最近一段时间一直在看`《ABP的开发指南》（基于DDD的经典分层架构思想）`。因为之前一段时间刚看完`《领域驱动设计：软件核心复杂性应对之道》`，概念比较多，看着有点空。于是拿起了这本书。应该说是不是书， 只是一个PDF版的开发指南。于是乎，就开始了。好了，废话不多说，首先是ABP领域层的结构介绍，如下图所示:

> ![](http://ww1.sinaimg.cn/large/006tKfTcgy1ff8aw3athrj30lo07gdh4.jpg)

从图中可以看到，`ABP`的领域层分为 `实体`，`仓储`，`工作单元`，`数据过滤器`，以及`领域事件`五个部分。这五个部分的功能作用，如果看过了解过`DDD`的应该不会陌生。接下来我将一一介绍这五个部分的详情，每个部分具体作用，以及实现。顺便给自己温习温习。附加一点自己粗浅的理解,如有不正确的地方，欢迎指正，交流。

## 2. 各个模块

###   2.1 ABP领域层-实体

> 很多对象不是通过它们的属性定义的，而是通过一连串的连续性事件和标识定义的。———摘录自《领域驱动设计》第5章 5.2 模式：Entity（又称为Reference Object）

 在ABP的实体中，实体继承至Entity。可以细分为下面三个部分。`1、实体类`，2、`接口约定`以及`3、IEntity接口`![](http://ww4.sinaimg.cn/large/006tNc79gy1ff9njn3u8gj30nm0cy0v2.jpg)

#### 2.1.1 首先是第一个`实体类`

查看源码发现如下：

```c#
namespace Abp.Domain.Entities
{
  /// <summary>
  /// A shortcut of <see cref="T:Abp.Domain.Entities.Entity`1" /> for most used primary key type (<see cref="T:System.Int32" />).
  /// </summary>
  [Serializable]
  public abstract class Entity : Entity<int>, IEntity, IEntity<int>
  {
  }
}
```

然后最后我们看下`IEntity<TPrimary>` 之后就真相大白了。

```C#
namespace Abp.Domain.Entities
{
  /// <summary>
  /// Defines interface for base entity type. All entities in the system must implement this interface.
  /// </summary>
  /// <typeparam name="TPrimaryKey">Type of the primary key of the entity</typeparam>
  public interface IEntity<TPrimaryKey>
  {
    /// <summary>Unique identifier for this entity.</summary>
    TPrimaryKey Id { get; set; }

    /// <summary>
    /// Checks if this entity is transient (not persisted to database and it has not an <see cref="P:Abp.Domain.Entities.IEntity`1.Id" />).
    /// </summary>
    /// <returns>True, if this entity is transient</returns>
    bool IsTransient();
  }
}
```

相当于每个继承于 `Entity` 的都有一个`Id`属性。`Id`属性是该实体的主键，所以，Id是所有继承自`Entity`类的实体的主键。当然，从源码中，我们可以发现。很显然这个Id数据的类型可以被更改。因为是`<TPrimary>`类型的，所以，你如果想要改成`long`,或者`Guid`等等啦。只需要向如下这样定义：

```C#
public class ClassA : Entity<Guid>
  {
    //...
  }
```

#### 2.1.2 第二个是`接口约定`

刚开始听到这个名字时，我觉得有点难以理解。但是在文章中，听了一下解释。可以帮助理解,如下：

>很多实体会有像`CreationTime` 用来指示该实体是什么时候被创建的。ABP提供了一些有用的接口来实现类似的功能，只需要实现这些接口的实体，就能够实现指定的功能。

在`接口预定`中,又分为如下三个部分:

![](http://ww3.sinaimg.cn/large/006tNc79gy1ff9o875maxj30hu09ign0.jpg)

首先是`审计（Auditing）`,这个接口主要的是`CreationTime` 这个属性。只要实体类实现`IHasCreationTime` 接口，当该实体被插入到数据库时，ABP会自动设置该属性的值为当前时间。

```C#
public interface IHasCreationTime{
  DateTime CreationTime{get;set;}
}
```

审计中还有一个接口`ICreationAudited` ，这个接口是扩展至`IHasCreationTime` ，并且该接口还有一个属性`CreatorUserId` ，可以用来记录当前用户的`Id`,

```C#
public interface ICreationAudited : IHasCreationTime{
  long ? CreatorUserId{get;set;}
}
```

`CreationAuditedEntity` 类已实现了` ICreationAudited` 我们可以直接继承来使用。

审计中还有一个实现类似修改功能的接口`IModificationAudited` 

```C#
public interface IModificationAudited{
  DateTime? LastModificationTime{get;set;}
  ling? LastModifierUserId{get;set;}
}
```

当然，如果你想都实现这些审计属性，ABP也提供给你了。`IAudited` 接口。

```C#
public interface IAudited : ICreationAudited, IModificationAudited{
  
}
```

可能你有点乱了，我来给你理理。大概就是如下这样的。

![](http://ww3.sinaimg.cn/large/006tNc79gy1ff9ozfeewxj30s20l2q6v.jpg)

这样简单吧。

接下来是`软删除`, 表示标记了一个实体已经被删除了。而不是从数据库中删除记录。对应的接口为`ISoftDelete`：

```C#
public interface ISoftDelete{
  bool IsDeleted{get;set;}
}
```

> 来源于文中：当一个实现了软删除的实体正在被删除，ABP会察觉到这个动作，并且阻止其删除，设置 `IsDeleted` 属性值为 `true`  并且更新数据库中的实体。

和审计中的`ICreationAudited`类似， 记录谁删除了这个实体。`IDeletionAudited` 接口。

```C#
public interface IDeletionAudited: ISoftDelete{
  long? DeleterUserId{get;set;}
  DateTime? DeletionTime{get;set;}
}
```

当然，如果你想要实现所有（包括，创建，修改，和删除。）那就实现这个终极接口， `IFullAudited`

```C#
public interface IFullAudited: IAudited , IDeletionAudited{
  //...
}
```

实体类 `FullAuditedEntity ` 实现了` IFullAudited` 接口。可以直接继承使用。

那么审计的终极总结图就如下：

![](http://ww1.sinaimg.cn/large/006tNc79gy1ff9pi17yyaj314k0o6jy5.jpg)

很清楚了吧！

PS：详情可以查看源代码 `Abp.Domain.Entities.Auditing`

#### 2.1.3最后一个是IEntity接口

 在2.1.1 中已经介绍了一下，这里就不赘述。`Entity` 实现了 `IEntity`接口（和`Entity<TPrimaryKey>` 实现了 `IEntity<TPrimaryKey>` 接口）。





###   2.2 ABP领域层-仓储



###   2.3 ABP领域层-工作单元



###   2.4 ABP领域层-数据过滤器



###   2.5 ABP领域层-领域事件





## 3. 结语

  本想一次性把这些都总结整理一下，突然发现内容有点多。还是一步一步来，慢慢整理把。这次先整理实体。好尴尬。

## 4.附录

图片来源：[Github_ABPChina](https://github.com/ABPFrameWorkGroup/AbpDocument2Chinese)

参考链接：

* https://www.aspnetboilerplate.com/Pages/Documents
* http://www.cnblogs.com/farb/p/ABPTheory.html
* http://www.cnblogs.com/mienreal/p/4528470.html





