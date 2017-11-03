# EntityFramework Core 学习系列（一）Creating Model

## Getting Started

**使用Command Line 来添加 Package**

&emsp;`dotnet add package Microsoft.EntityFrameworkCore.SqlServer`  使用 `-v` 可以指定相应包的版本号。

**使用`dotnet ef` 命令**

&emsp;需要在`.csproj` 文件中包含下面引用

```c#
<ItemGroup>
	<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0"/>
</ItemGroup>
```

## Creating a Model

**Fluent API**

&emsp;在继承至 `DbContext` 的子类中，重载 OnModelCreating() 方法进行Fluent API 的配置。

```c#
public class MyDbContext : DbContext
{
	//...
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
    	modelBuilder.Entity<TEntity>()
    		.Property(b => b.Url)
    		.IsRequired();
    	//...
    }
    //...
}
```

**Data Annotation**

&emsp;或者可以使用数据注解直接在实体中进行配置：

```c#
public class Blogs
{
  	public string BlogName { get; set; }
  	[Required]
  	public string Url { get; set; }
}
```

关于配置的顺序规范，

> Fluent API > Data Annotations > Conventions

### Include & Exclude

&emsp;有下列三种情况类或实体会被包含：

1. By convention, types that are exposed in `DbSet` properties on your context are included in your model.
2. Types that are mentioned in the `OnModelCreating` method are also included. 
3. Any types that are found by recursively exploring the navigation properties of discovered types are also included in the model.

你可以通过 Data Annotation 或者 Fluent API 来 排除包含，示例如下：

```c#
//Data Annotations Example Below:
public class Blogs
{
  public int BlogId { get; set;}
  
  public BlogAuthor BlogAuthors { get; set;}
}

[NotMapped]
public class BlogAuthor
{
  public string FirstName { get; set;}
  //...
}

//Fluent API Example Below:
public class MyDbContext : DbContext
{
	public DbSet<Blog> Blogs { get; set;}
	
	protected override OnModelCreating(ModelBuilder modelBuilder)
    {
    	modelBuilder.Ignore<BlogAuthor>();
    }
}
public class Blogs
{
  public int BlogId { get; set;}
  
  public BlogAuthor BlogAuthors { get; set;}
}

public class BlogAuthor
{
  public string FirstName { get; set;}
  //...
}
```

当然除了可以排出/包含类之外， 还可以自己配置相应的属性如下：

```C#
//Data Annotations Example Below:
public class Blogs
{
  public int BlogId { get; set;}
  
  public BlogAuthor BlogAuthors { get; set;}
  
  [NotMapped]
  public DateTime BlogAddedTime { get; set;}
}

//Fluent API Example Below:
public class MyDbContext : DbContext
{
	public DbSet<Blog> Blogs { get; set;}
	
	protected override OnModelCreating(ModelBuilder modelBuilder)
    {
    	modelBuilder.Entity<Blog>()
    		.Ignore(b => b.BlogAddedTime);
    }
}
public class Blogs
{
  public int BlogId { get; set;}
  
  public BlogAuthor BlogAuthors { get; set;}
  
  public DateTime BlogAddedTime { get; set;}
}

```

### Key

&emsp;关于EF Core 中的Key，	按照规范，如果一个属性命名为Id  或者 以Id结尾的都会被配置成该实体的主键。比如下面的这个：

```c#
public class TestClass
{
  public int Id { get; set; }
  
  //or
  public int MyId { get; set; }
}
```

或者你可以按照 Data Annotation 或者 Fluent API 来进行配置：

```C#
// Data Annotations
public class Car 
{
  [Key]
  public string CarLicense { get; set; }
  
  public string CarFrameCode { get; set;}
}

//Fluent API
//...
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
  modelBuilder.Entity<Car>()
    	.HasKey(c => c.CarLicense);
}
//...

// Multiple Properties to the key ,复合主键的配置只能用Fluent API
modelBuilder.Entity<Car>()
  	.HasKey(c => new { c.CarLicense, c.CarFrameCode });

```

### Generated Values

&emsp;By convention, primary keys that are of an integer or GUID data type will be setup to have values generated on add. All other properties will be setup with no value generation.

