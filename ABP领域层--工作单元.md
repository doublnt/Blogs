[TOC]

## 1. 前言

  在上一篇博文中（http://www.cnblogs.com/xiyin/p/6842958.html） 我们讲到了ABP领域层的仓储。这边博文我们来讲 工作单元。个人觉得比较重要。文章的大致结构分为六部分，如下图所示：

![](http://ww2.sinaimg.cn/large/006tKfTcgy1ffpse70v77j31ii0faaem.jpg)



## 2. 工作单元

### 2.1 通用连接和事务管理方法

---

 `.NET` 使用连接池（`connection pooling`）。因此，创建一个连接实际上是从连接池中取得一个连接，因为创建新连接会有成本。如果没有任何连接存在于连接池中， 一个新的连接对象会被创建并且添加到连接池中。当你释放连接，它实际上是将这个连接对象送回到连接池。这并不是实际意义上的释放。那么，最佳实践是什么呢？ 应该在使用完之后释放掉连接对象。

 那么在应用程序中，创建和释放连接的方法都有哪些呢？ 

1. 当`Web请求`时，创建一个连接（`Application_BeginRequest`位于 `global.asax`中的事件），使用同一个连接对象处理所有的数据库操作，并且在请求结束的时候关闭／释放这个连接（`Application_EndRequest`事件）。

2. 创建一个连接当需要的时候（只要在使用它之前）并且释放它在使用它之后。

   上面两个方法的优缺点呢？第一种简单却效率低下，为何？

   > * 可能这个`Web请求`不需要操作数据库，但是连接却会开启。这对于连接池来说是个毫无效率的使用方式。
   > * 会让`Web请求`的运行时间变长，并且数据库操作还会需要一些执行。
   > * 同样的这是一个使用事务式的数据库操作的最佳场景。如果又一个操作发生失败，所有的操作都会会滚。因为事务会锁住数据库中的一些数据列（事件数据表），它必定是短暂的。

   第二种虽然高效，但是你需要不断的进行创建和释放连接操作。

### 2.2 ABP的连接和事务管理

---

 上面讲了两个连接管理的方法，那么ABP的连接管理方法又是怎么样的呢？ABP结合上面两个连接管理的方法。如下：

#### 2.2.1 仓储类

 仓储是主要的数据库操作的类。ABP开启了一个数据库连接并且在进入到仓储方法时会启用一个事务。因此， 你可以安全地使用连接于仓储方法中。在仓储方法结束后，事务会被提交并且释放掉连接。例如仓储方法抛出任何异常，事务会被会滚并且释放掉连接。在这个模式中，仓储方法是单元性的（相当于一个工作单元 `unit of work`）。

 当仓储方法调用另一个仓储方法（一般来说，若工作单元方法调用另一个工作单元方法），都使用同一个连接和事务。第一个被调用到的仓储方法负责管理连接和事务，而其余被它调用的仓储方法则只当初使用，不管理。

#### 2.2.2 应用服务

 一个应用服务的方法也被考虑使用工作单元。例：

```C#
public class TestService : ITestService
{
  private readonly ITestRepository _testRepository;
  private readonly IStaticRepository _staticRepository;
  
  public TestService(ITestRepository testRepository,
                    IStaticRepository staticRepository)
  {
     _testRepository = testRepository;
     _staticRepository = staticRepository;
  }
  
  public void CreateTestDemo(TestDemo input)
  {
    var test = new Test{ Name = "Robert" ,Age ="111"};
    _testRepository = testRepository;
    _staticRepository = staticRepository;
  }
}
```

在`CreateTestDemo` 方法中，新增一个`Test` 实体使用了`Test` 仓储和`Static` 仓储。两个仓储共享同一个连接和事务。ABP开启一个数据库连接并且开启一个事务于进入到 `CreateTestDemo`前 ，若没有任何异常抛出，接着提交这个事务于方法结尾时。若有异常抛出，则会回滚这个事务。在这种机制下，所有数据库操作都成了单元性的。

#### 2.2.3 工作单元

 工作单元在后台替仓储和恶应用服务的方法工作，加入你想要控制数据库的连接和事务，你就需要直接操作工作单元。

1.示例一：

```C#
[UnitOFWork]
public void CreateTest(Test input){
  var test = new Test {Name = "Robert", Age=111 };
  _testRepository.Insert(test);
  _staticRepostiory.IncreaseAge();
}
```

`CreateTest `方法转变成工作单元并且管理数据库连接和事务，两个仓储对象都使用相同的工作单元。**如果是应用服务的方法，则不需要添加 UnitOfWork 属性**

### 2.3 工作单元

#### 2.3.1 禁用工作单元

 你或许想要禁用应用服务方法的工作单元（默认是开启的），只需在属性上设置`IsDisable` 属性即可：

```C#
[UnitOfWork(IsDiable = true)]
public virtual void Remove(Test input){
  _removeRepository.Remove(input.Id);
}
```

平常，一般不需要这样做。因为应用服务的方法都应该是单元性的，并且使用数据库。

#### 2.3.2 无事务的工作单元

 工作单元默认是具有事务性的，因此，ABP启动／提交／回滚一个显性的数据库等级的事务。在有些特殊案例中，事务可能会导致问题。因为它可能会锁住有些数据列或是数据表于数据库中。在此这些情景下，你或许会想要禁用数据库等级的事务。`UnitOfWork` 属性可以如下写法，取得一个布尔值来让他的如非事务型工作单元工作。

```C#
[UnitOfWork(false)]
public GetTasksOutput GetTasks(GetTaskInput input){
  var tasks = _taskRepository.GetAllWithPeople(input.Id ,input.Age);
  return new GetTasksOutput{
    Tasks = Mapper.Map<List<TaskDto>>(tasks)
  };
}
```

`[UnitOfWork(false)] `等同于` [UnitOfWork(isTransaction:false)]`

**如果你已经位于事务性工作单元区域内，设定 `isTransaction` 为 `false` 这个动作会被忽略。**

#### 2.3.3 工作单元调用其他工作单元

 若工作单元方法（一个有`UnitOfWork` 属性的方法）调用另一个工作单元方法，他们共享同一个连接和事务。第一个方法管理连接，其他的方法只是使用它。这在所有方法都执行在同一个线程下是可行的（或者是在同一个`Web请求`中）。实际上，当工作单元区域开始，,所有的程序代码都会在同一个线程中执行并共享同一个连接事务, 直到工作单元区域终止。这对于使用 `UnitOfWork` 属性和 `UnitOfWorkScope` 类来说都是一样的。如果你创建了一个不同的线程/任务,它使用自己所属的工作单元。

#### 2.3.4 自动化的saving changes

 当我们使用工作单元到方法上,ABP 自动的储存所有变化于方法的末端。假设我们需要一个可更新` person `名称的方法：

```C#
[UnitOfWork]
 public void UpdateName(UpdateNameInput input) {       
   var person = _personRepository.Get(input.PersonId); 
   person.Name = input.NewName;    
 } 
```

这样，名称就被修改了。`ORM 框架`会持续追踪实体所有的变化于工作单元内,且反映所有变化到数据库中。

#### 2.3.5 仓储接口的GetAll 方法

 你在仓储方法外调用` GetAll `方法, 这必定得有一个开启状态的数据库连接,因为它返回` IQueryable` 类型的对象。这是需要的,因为` IQueryable `延迟执行。它并不会马上执行数据库查询,直到你调用 `ToList()`方法或在 `foreach `循环中使用`IQueryable`(或是存取被查询结果集的情况下)。 因此,当你调用 `ToList()`方法,数据库连接必需是启用状态。 

如`GetAll()`方法,如果需要数据库连接且没有仓储的情况下,你就必须要使用工作单元。注意,应用服务方法默认就是工作单元。

#### 2.3.6 工作单元属性的限制

你可以在下面的情景下使用` UnitOfWork `属性标签。

* 类所有`public `或`public virtual` 这些基于界面的方法（就像是应用服务基本服务界面）
* 自我注入类的`public virtual `方法（像是`MVC Controller` 和`WebApi Controller`）
* 所有`protected virtual` 方法

> 建议将方法标示为` virtual`。你无法应用在`private `方法上。因为，ABP使用`dynamic proxy` 来实现，而私有方法就无法使用继承的方法来实现。当你不使用依赖注入且自行化初始化类，那么 `UnitOfWork `属性就无法正常运作。



### 2.4 选项

 我们可以在 `startup configuration` 中改变左右工作单元的所有默认值。这通常是用了我们模块中的`PreInitialize `方法来实现。

```C#
public class SimpleTaskSystemCoreMudule : AbpModule{
  public override void PreInitialize(){
    Configuration.UnitOfWork.IsolationLevel = IsolationLEvel.ReadCommitted;
    Configuration.UnitOfWork.Timeout = TimeSpan.FromMinutes(30);
  }
}
```



### 2.5 方法

 工作单元系统运作是无缝且不可视的。但是，在有些特例下， 你需要调用它的方法。

**SaveChanges**

 ABP储存所有的变化于工作单元的尾端，你不需要做任何事情。但是，有些时候，你或许会想要在工作单元的过程中就储存所有变化。在这个案例中，你可以注入 `IUnitOfWorkManager` 并且调用 `IUnitOfWorkManager.Current.SaveChanges() `方法。



### 2.6 事件

 工作单元具有 `Completed/Failed/Disposed` 事件。 你可以注册这些事件并且进行所需的操作。注入` IUnitOfWorkManager` 并且使用 `IUnitOfWorkManager.Current` 属性来取得当前已激活的工作单元并且注册它的事件。 

你或许会想要执行有些程序代码于当前工作单元成功地完成。示例:

```C#
public void CreateTask(CreateTaskInput input){
  var task = new Task { Description = input.Description };
  if(input.AssignedPersonId.HasValue){
    task.AssignedPersonId = input.AssignedPErsonId.Value;
    _unitOfWorkManager.Current.Completed += (sender,args) =>{
      //....
    };
  }
  _taskRepository.Insert(task);
}
```









