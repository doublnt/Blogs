# Repository 模式以及Unit of Work 模式的理解与使用

## 前言

&emsp;在以前的[博文](http://www.cnblogs.com/xiyin/p/6384702.html)中，当时在使用DDD时就有一点疑惑，整理了一些 Reposiotry 模式的相关文章，但是没有深入去理解，导致最近在写代码时在思考到底为什么要用Repository 模式？，有人说EF已经实现了Repository 模式，你为什么又要自己去实现了?（其中这篇[文章](http://www.cnblogs.com/leotsai/p/entity-framework-doesnt-need-additional-repository.html)中更是强烈抨击了EF中使用Repository模式，我持保守态度。）当然，与之对应的就是 Unit Of Work模式。本文就是来谈谈这两种模式的理解以及使用。

## Repository

&emsp;**理解某种模式最重要的就是知道这个模式到底是来解决什么问题的**，那么Repository是解决什么问题的呢？在MSDN的官网上的一篇[文章](https://msdn.microsoft.com/en-us/library/ff649690.aspx)我们可以看到其介绍:

> The repository mediates between the data source layer and the business layers of the application. It queries the data source for the data, maps the data from the data source to a business entity, and persists changes in the business entity to the data source. A repository separates the business logic from the interactions with the underlying data source or Web service. 
>
> Repositories are bridges between data and operations that are in different domains. 

## Unit Of Work

&emsp;工作单元解决的基本问题是记录操作过的各种对象，以便知道为了使内存中的数据与数据库同步需要考虑哪些对象。

&emsp;把每个所改变的对象加上“脏”标志的做法要比把对象保存在变量中好。在事务处理完后，需要找出所有加了“脏”标记的对象并把它们写入到数据库中。

&emsp;引发你去处理数据库的明显原因是变化：创建新对象和更新或删除已经存在的对象。工作单元就是 一个记录这些变化的**对象**。只要开始做一些可能会对数据库有影响的操作，就创建一个工作单元去记录这些变化。每当创建、改变或者删除一个对象时，就通知此工作单元。

&emsp;Repository 主要解耦了数据访问层以及业务逻辑层。接下来建立一个简单的图书模型来演示我们的模式。

```C#
首先是Book实体：
public class Book
    {
        public int Id { get; set; }
        public string BookName { get; set; }
        public string Publisher { get; set; }
        public double Price { get; set; }
        public Author Authors { get; set; }
    }

然后就是Author实体：
public class Author
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }
```

然后我们先用EntityFramework来简单的进行数据的增/删操作。（使用的数据库为SQLite）

```C#
		using (var db = new BookDbContext())
            {
                if (!db.Books.Any())
                {
                    db.Books.AddRange(new Book
                    {
                        Id = 1,
                        BookName = "Python In Action",
                        Price = 1.1,
                        Publisher = "China Publisher"
                    });
                    db.SaveChanges();
                }
                if (!db.Authors.Any())
                {
                    db.Authors.AddRange(new Author
                    {
                        Id = 1,
                        Name = "Xi Yin",
                        Age = 15
                    });
                    db.SaveChanges();
                }

                var books = db.Books
                    .FirstOrDefault(p => p.BookName != null);

                var author = db.Authors.
                    FirstOrDefault(p => p.Id == 1);

                books.Authors = author;
                
                Console.WriteLine(books.BookName + "\n" + books.Price + "\n" +
                books.Publisher + "\n" + books.Authors.Name);

            }
```

&emsp;在上述代码中我们可以看到，每次我给一个实体添加数据时，或者更新数据是，都需要 调用一次 `SaveChanges` 来持久化到数据库中。

## Repository 和Unit Of Work的使用

&emsp;接下来先讲讲Repository模式以及Unit Of Work 的使用：



**小记：如果项目中无法使用Include 属性，则需要引用 `Microsoft.EntityFrameworkCore`**



## 参考：

[Repository](https://msdn.microsoft.com/en-us/library/ff649690.aspx)



`dotnet ef migrations add InitialCreate`

`dotnet ef database update`
