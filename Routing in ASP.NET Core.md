##Routing in ASP.NET Core

### 前言

&emsp;这篇文章是关于 ASP.NET Core(下文中用AEC简写表示 ASP.NET Core) 中的路由的。主要是从下面两个AEC 官方文档上了解到的内容，再结合自己实践中得出的一些结论，以及一些建议和注意事项。

### 路由是什么

&emsp;首先讲讲AEC Mvc中的路由，AEC 中使用 Routing Middleware 即路由中间件来处理URL中的请求，然后映射到相应的Action中。官网的这张Middleware图感觉挺不错。表示了路由的请求以及相应的逻辑。More about [Middleware](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware)![](https://ws2.sinaimg.cn/large/006tNc79gy1fj7p8e40uij30go0ao3z9.jpg)

路由一般定义在Startup.cs 中，或者直接以 Attribute 形式定义在 Controller 中，这两种方法可以混合起来使用，因为两种各有偏好，比如在设计WebApi的时候，可能用属性来定义路由会比较多。向下面这样：

```c#
[Route("Hello")]
[HttpGet()]
public IActionResult Index(){
    //return View or something.
}
```



### 如何使用，创建你的路由

&emsp;接下来就讲讲如何创建路由，一般我们都是在 Starup.cs 的 Configure 的 app.UseMvc 中定义创建的。就像下面这样，

```c#
//Startup.cs
app.UseMvc(options => 
{
   options.MapRoute("default","{Controller=Home}/{Action=Index}/{Id?}") 
});

//HomeController.cs

public Class HomeControler
{
  public string Index(int? id)
  {
    return $"Hello,Robert!+{id}";
  }
}
```

这样我们就创建好了一个路由，上面的匹配的URL就像这样 `/Home/Index/1`或者 `Home/Index` 或者 `/Home`。因为是默认路由，所以我们比如网页上直接输入`localhost:5000`的话，就直接导航到这个地方了。

如下图所示：

![](https://ws4.sinaimg.cn/large/006tNc79gy1fj7wljo6a1j30hs0i80ut.jpg)

如果感觉上面的那个写法有点复杂，而且你的项目又只有一个路由的话，你可以直接这样

```c#
app.UseMvcWithDefaultRoute();
//Equals to below 
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

说路由是用 Middleware 的，那么怎么用起了 `UseMvc` 呢，当我们在使用UseMvc时，就相当于下面这样

```c#
var routes = new RouteBuilder(app);

routes.DefaultHandle = new MvcRouteHandler(..);

routes.MapRoute(...);

//创建路由集合，并添加中间件
app.UseRouter(routes.Build());
```

如果你有多个路由，那么直接在 app.UseMvc 中再自己定义就可以了

```C#
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
   routes.MapRoute("First","..")
     //...
});
```



如果有下面一段代码：

```C#
public class TestController
{
    public string Test(int id)
    {
        return "Test1";
    }
  	public string Test(int id ,string s)
  	{
    	return "Test2";  
  	}
}
```

当我使用URL 为`/Test/Test`/11/ 你觉得结果会是什么？ 结果就是这个

```shell
Microsoft.AspNetCore.Mvc.Internal.AmbiguousActionException: Multiple actions matched. The following actions matched route data and had all constraints satisfied:
```

程序判断不出哪个到底是最符合的，难道就不能有同名的ActionName 吗？ 未必。接下来就要介绍另外一种 Attribute 形式的路由。

回到上面，我如果改成下面这样，那么就可以完美运行了,

```C#
public class TestController
{
    public string Test(int id)
    {
        return "Test1";
    }
  	[HttpPost]
  	public string Test(int id ,string s)
  	{
    	return "Test2";  
  	}
}
```

结果就是下图预想的这样

![](https://ws4.sinaimg.cn/large/006tNc79gy1fj7x89ys0vj30xc078dgg.jpg)

 

除了上面的方式，我们可以直接改写我们刚刚在使用 app.UseMvc() 的那种方式，改为用属性，如下所示：

```c#
public class HomeController
{
    [Route("")]
  	[Route("Home")]
  	[Route("Home/Index")]
  	public string Index(int? id){
        return "Hello, Robert!"
    }
}
```



我们可以使用同样名字的URL，但是导航到不同的地址，这可以通过Http Attribute来实现，如下图所示：

```C#
[HttpGet("/products")]
public IActionResult ListProducts()
{
  //...
}

[HttpPost("/products")]
public IActionResult CreateProduct(...)
{
    //...
}
```



还有一种情况就是在我们开发Api 的时候挺有用的，比如我想给我的路径前面都加上`/api/`

那么你可以直接在Class 上面定义属性

```c#
[Route("api")]
public class MyApiController
{
    //...
}
```



&emsp;当然，你也可以定义自己的属性路由，比如像下面这样，

```C#
public class MyTestControllerAttribute : Attribute,IRouteTemplateProvider
{
  public string Template => "test/[controller]";
  public int? Order {get;set;}
  public string Name{get;set;}
}
```

当[MyTestController] 使用时，Template 则自动设置为 `test/[controller]`

###注意事项

&emsp;需要注意的是，路由是有顺序的，这个顺序就是按照你定义时的顺序，当发起一个请求时，逐个遍历，找到后导航到对应的action，找不到则返回404。

### 参考链接

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing
* https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
* 建议多阅读官方英文原版文档，毕竟每个人的理解都是不一样的。