## 1. 前言

 在上一篇博文中http://www.cnblogs.com/xiyin/p/6810350.html 我们讲到了ABP领域层的实体，这篇博文继续讲ABP的领域层，这篇博文的主题是ABP领域层—仓储。我们在上篇博文中介绍的ABP领域层的大致结构，在这篇文章就不一一赘述了。详情可以查看[上篇博文](http://www.cnblogs.com/xiyin/p/6810350.html)。接下来直接进入主题。仓储的定义是这样的：

> 在领域层和数据映射层的中介，使用类似集合的接口来存取领域对象----Martin Fowler。

仓储被用于领域对象在数据库上的操作（实体`Entity`和值对象 `Value types`），一般来说，我们针对不同的实体（或聚合根`Aggregate Root`） 会创建相对应的仓储。

> 疑问：“那是不是对每个实体都创建相应的仓储呢？”

本文讲述的结构如下:

![](http://ww1.sinaimg.cn/large/006tNc79gy1ffhozka85lj31860e6tdr.jpg)

接下来，我将一一讲述。



## 2.ABP领域层—仓储

### 2.1 IRepository 接口

---

 在ABP中，仓储类要实现`IReposiotry`接口。最好的方式是针对不同仓储对象定义各自不同的接口。

如果你的实体`ID` 是`int` 类型的，那么可以使用如下定义：

```c#
public  interface IPersonREpository : IRepository<Person>
{
  //...
}
```

如果不是`int` 类型的，可以使用如下定义：

```C#
public interface IPersonRepository : IRepository<Person, long>
  {
    //...
  }
```

对于仓储类，`IRepository `定义了许多泛型的方法。比如`Insert`,`Select`,`Update`,`Delete`, 详情可以在源码的`Abp.Domain.Repositories` 程序集中详细查看。

接下来对` CRUD`进行一一的介绍。

#### 2.1.1 查询（Query）

---

 **取得单一实体：**

```C#
TEntity Get(TPrimaryKey id);
Task<TEntity> GetAsync(TPrimaryKey id);
TEntity Single(Expression<Func<TEntity,bool>> predicate);
TEntity FirstOrDefault(TPrimaryKey id);
Task<TEntity> FirstOrDefaultAsync(TPrimaryKey id); TEntity FirstOrDefault(Expression<Func<TEntity, bool>> predicate); Task<TEntity> FirstOrDefaultAsync(Expression<Func<TEntity, bool>> predicate); TEntity Load(TPrimaryKey id)
```

`Get`方法被用于根据主键值（`Id`）取得对应的实体。当数据库中根据主键值找不到相符合的实体时，它会抛出异常。

主要讲一下`Single `方法的实例，因为参数是一个表达式：

```C#
var person = _personRepository.Get(42);
var person = _personRepository.Single(p => p.Name == "Robert");
```

**Tip1:`Single`方法在给出的条件找不到实体或符合的实体超过一个以上时，都会抛出异常。**

**Tip2:`FirstOrDefault`也一样，但是当没有符合`Lambda` 表达式或`Id`的实体时，会返回`null`（取代抛出异常）。当有超过一个以上的实体符合条件，它只会返回第一个实体。**

**Tip3: `Load` 并不会从数据库中检索实体，但它会创建延迟执行所需的代理对象。如果你只使用`Id` 属性，实际上并不会检索实体，它只有在你存取想要查询实体的某个属性时才会从数据库中查询实体。**

***取得实体列表：**

```C#
List<TEntity> GetAllList(); Task<List<TEntity>> GetAllListAsync(); List<TEntity> GetAllList(Expression<Func<TEntity, bool>> predicate); Task<List<TEntity>> GetAllListAsync(Expression<Func<TEntity, bool>> predicate); IQueryable<TEntity> GetAll();
```

`GetAllList` 被用于从数据中检索所有实体，重载并且提供过滤实体的功能，

```C#
var allPeople = _personRepository.GetAllList();
var somePeople = _personRepository.GetAllList(person => person.IsActive && person.Age > 20 );
```

`GetAll `返回`IQueryable<T>`类型的对象。因为是`IQueryable` 类型的，所以我们可以在调用完成后，进行`Linq `操作。相关的`Linq `操作函数，可以查看我的[这篇博文](http://www.cnblogs.com/xiyin/p/6086119.html)。示例：

```C#
var query = from person in _personRepository.GetAll()
  where person.IsActive
  orderby person.Name
  select person;
var people = query.ToList();

List<Person> personList2 = _personRepository.GetAll().Where(p =>p.Name.Contains("H")).OrderBy(p => p.Name).Skip(40).Take(20).ToList();
```

***自定义返回值**

ABP 有一个额外的方法来实现`IQueryable<T>` 的延迟加载效果，而不需要在调用的方法上添加`UnitOfWork `这个属性卷标。

```C#
T Query<T> (Func<IQueryable<TEnity>,T> queryMethod);
```

查询方法接受 `Lambda `（或一个方法）来接受`IQueryabel<T>` 并且返回任何对象类型。示例：

```C#
var person = _personRepository.Query(q => q.Name.Contains("H").OrderBy(p => p.Name).ToList());
```

#### 2.1.2 新增

---

 `IRepository` 接口定义了如下方法来新增一个实体到数据库。

```C#
TEntity Insert(TEntity entity); Task<TEntity> InsertAsync(TEntity entity); TPrimaryKey InsertAndGetId(TEntity entity); Task<TPrimaryKey> InsertAndGetIdAsync(TEntity entity); TEntity InsertOrUpdate(TEntity entity); Task<TEntity> InsertOrUpdateAsync(TEntity entity); TPrimaryKey InsertOrUpdateAndGetId(TEntity entity); Task<TPrimaryKey> InsertOrUpdateAndGetIdAsync(TEntity entity);
```

**Tip:**新增方法会新增实体到数据库并且返回相同的已新增实体。`InsertAndGetId `方法会返回新增实体的标识符（`Id`）。当我们采用自动递增标识符值且需要取得实体的新产生标志符值时非常好用。

#### 2.1.3 更新

---

```C#
TEntity Update(TEntity entity);
Task<TEntity> UpdateAsync(TEntity entity)
```

#### 2.1.4 删除

---

```C#
void Delete(TEntity entity); 
Task DeleteAsync(TEntity entity); 
void Delete(TPrimaryKey id); 
Task DeleteAsync(TPrimaryKey id);
void Delete(Expression<Func<TEntity, bool>> predicate); 
Task DeleteAsync(Expression<Func<TEntity, bool>> predicate); 
```

#### 2.1.5 其他方法

---

```C#
int  Count(); 
Task<int> CountAsync(); 
int  Count(Expression<Func<TEntity, bool>> predicate); 
Task<int> CountAsync(Expression<Func<TEntity, bool>> predicate); Long  LongCount(); 
Task<long> LongCountAsync(); 
Long  LongCount(Expression<Func<TEntity, bool>> predicate); Task<long> LongCountAsync(Expression<TEntity, bool>> predicate)
```



### 2.2 仓储的实现

---

 ABP在设计上是采取不指定`ORM`框架或其它存取数据库技术的方法。只要实现`IRepository `接口，任何框架都可以使用。

 当你使用`NHIbernate` 或`EntiyFramework`，如果提供的方法已足够使用，你就不需要为你的实体创建仓储对象了。我们可以直接注入`IRepository<TEntiy>`或`IRepository<TEntity,TPrimaryKey>` 下面实例为`application service `使用仓储来新增实体到数据库：

```C#
public class PersonAppService : I PersonAppService
{
  	private readonly IRepository<Person> _personRepository;
  
  	public PersonAppService(IRepository<Person> personRepository)
  	{
    	_personRepository = personRepository;
  	}
  
 	public void CreatePerson(CreatePersonInput input)
    {
      person = new Person {Name = input.Name , EmailAddress = input.EmailAddress};
    }
	_personRepository.Insert(person);
}
```



### 2.3 管理数据库连接

---

 **数据库连接的开启和关闭，在仓储方法中，ABP会自动化的进行连接管理。**

 当仓储方法被调用后，数据库连接会自动开启且启动事务。当仓储方法执行结束并且返回以后，所有的实体变化都会被储存，事务被提交并且数据库连接被关闭，一切都由ABP自动化的控制。如果仓储方法抛出任何类型的异常，事务会自动地回滚并且数据连接会被关闭。

 如果仓储方法调用其他仓储方法（即便是不同的仓储的方法），它们共享同一个连接和事务。连接会由仓储方法调用链最上层的那个仓储方法所管理。



### 2.4 仓储的生命周期

---

 所有的仓储对象都是暂时性的，这就是说，它们是在由需要的时候才会被创建。ABP大量的使用依赖注入，当仓储类需要被注入的时候，新的类实体会由注入容器自动地创建。

### 2.5 仓储的最佳实践

---

* 对于一个T类型的实体，是可以使用`IRepository<T>` 。但别任何情况下都创建定制化的仓储，除非我们真的很需要。预定于仓储方法已经足够应付各种案例。
* 创建定制的仓储可以实现`IRepository<TEntity>`
* 仓储类应该是无状态的。这意味着，你不该定义仓储等级的状态对象并且仓储方法的调用也不应该影响到其他调用。
* 当仓储可以使用像 依赖注入，尽可能较少，或不根据其他服务。



## 3. 结语

 正好让我回顾回顾之前阅读的书。虽说都是基础内容，但也加深了其理解。

## 4.参考文献

* https://www.aspnetboilerplate.com/Pages/Documents