## 1. 前言

 感觉有好长时间没有接触`View` 了，周末闲来无事，翻翻书桌上的书来回顾回顾`ASP.NET MVC`中`View`的相关内容。

## 2. View概述

 `View` 通过应用程序在`Action` 中返回 `ViewResult` 或`PartialViewResult `，在运行阶段内部调用`ExecuteResult` 方法而产生模版转换（`transforamtion`），将运行阶段运算后产生的Model 经由模版引擎（`template engine`） 进行转换，从而生成HTML页面代码，输出到浏览器。

**_ViewStart.cshtml:**

```html
@{
  Layout = "~/Views/Shared/_Layout.cshtml";
}
```

**`_ViewStart.cshtml`的默认`Layout` 行为并不适用于 `Partial View` 部分试图，`Partial View `不会引用这项指定。**

### 2.1 Layout 的结构

---

 下面是Layout 的示例代码：

```html
<!--Layout 示例-->
<html>
<head>  
  <titile>@ViewBag.Title</titile>
  @RenderSection("head")
</head>  
<body>
  <div class ="header">
    <!--这里放header 的内容-->
  </div>
  <div class ="main">
    <div class = "aside">
      <!--这里放侧边栏拉或者菜单等等-->
      @RenderSection("sideBar", required:false)
    </div>
    <div class = "content">
      @RenderBody();
    </div>
  </div>
  <div class= "footer">
    <!--这里放页面底部的内容-->
  </div>
  @RenderSection("Script",required:false)
</body>
</html>
```

### 2.2 RenderSection

---

`@RenderSection`具有一个必要参数作为区域名称，并且有一个可选参数`required`， 省略`required` 参数或者设置`required:true `用来指定套用这份`Layout `的`View`是否必须满足这个区域，如果没有提供该区域，会报错。

```html
<!--@RenderSection示例-->
@{
  Layout = "~/Views/Shared/_Layout.cshtml";
}
@section head{
  <meta name="description" content = "This is a test View By Robert."/>
}
@section sideBar{
<h2>
  Hello, Are you Ok?
</h2>
<ul>
  <li>
    <a href ="http://www.cnblogs.com">cnblogs</a>
  </li>  
  <li>
    <a href = "http://q.cnblogs.com">q.cnblogs.com</a>
  </li>
</ul>
}
hahahhaha
```

### 2.3 Layout 与View 的执行顺序 

---

 **在执行顺序上，`View` 会被优先执行，然后被`View`引用的`Layout` 执行**，这一点经常会被误解为相反的情况。例如在程序中以`ViewBag`、`ViewData`在`View` 与`Layout` 之间进行变量的传递，就会有执行顺序的问题需要考虑，因此一开始需要特别留意。



## 3 View 获取数据的方法

下面用一个表来展现几种传递数据的方式：

|    方式    |                  优点                  |               缺点                |
| :------: | :----------------------------------: | :-----------------------------: |
|  Model   |          强类型，得以在编译期间进行类型检查           | 当以string 类型作为ViewModel 时会有一点小麻烦 |
| ViewData |       不需要建立ViewModel 类即可进行传递数据       |          弱类型，无编译期间类型检查          |
| ViewBag  |       不需要建立ViewModel 类即可进行传递数据       |          弱类型，无编译期间类型检查          |
| TempData | 不需要建立ViewModel 类即可进行传递数据，可以跨Action传递 |          弱类型，无编译期间类型检查          |

`Model`类型就不多说了，接下来讲一下 `ViewBag` ，`ViewData`以及`TempData`

### 3.1 ViewData 和ViewBag

 `ViewData` 是`Controller` 实例中的属性，类型为`ViewDataDictionary`， `ViewDataDictionary` 实现了`IDictionary<string,object>` , 将`Dictionary`中的`TValue `泛型参数声明为`object`，故可以放置任意类型数据。

```csharp
//赋值
ViewData["Now"] = DateTime.Now;

//使用
@ViewData["Now"]

//读取
var dateTime = (DateTime)ViewData["Now"];
```

 `ViewBag `是`Controller` 实例中的属性，类型为`dynamic`， 与`ViewData`性质完全相同，只是使用的机制稍有不同。

```c#
//赋值
ViewBag.Now = DateTime.Now;

//使用
@ViewBag.Now

//读取
var dateTime = (DateTime) ViewBag.Now;
```

若以“`@ViewBag.自定义属性`” 或 “`@ViewData["自定义属性"]`”的方式在页面中呈现内容，是不需要考虑类型转换的。由于都会以`ToString `方法强制为字符串输出，因此在只是输出的简单情况下不必编写复杂的类型转换程序代码。

### 3.2 TempData

 `TempData` 是`Controller` 实例中的属性，类型为`TempDataDictionary`， `TempDataDictionary` 实现了`IDictionary<string,object>`。

 各种特性让TempData 行为看起来与`ViewData` 、` ViewBag` 相似，但实际上存放在`TempData `的数据只要经过一次读取就会消失在容器中，另一个特性是如果数据只存不取，那么这份数据的默认有效期与`ASP.NET Session` 一样长，未经调整的`Session` 有效期为20分钟。在`ASP.NET MVC `内部的原因是`TempData `通过内部的`SessionStateTempDataProvider` 实现将暂存数据放置于`Session` 中。



## 4 Layout 嵌套

比如下面这张图，（画的略丑）

![](http://ww2.sinaimg.cn/large/006tNc79gy1ffjkqevk08j31760vundj.jpg)

那么我用Layout 的代码如何设计呢？如下所示：

```html
<!--_Layout.cshtml-->
<html>
  <head>
    <link rel = "stylesheet" href ="~/Content/test.min.css"/>
    @RenderSection("head",required:false)
  </head>
  <body>
    <div class = "header">
      <!-- 放Logo 啥的-->
    </div>
    <div class = "content">
      @RenderBody();
    </div>
    <div class = "footer">
      <!--页面底部-->
    </div>
    <script src = "~/Scirpts/test.min.js"></script>
    @RenderSection("scripts",required:false)
  </body>
</html>
```

接下来是_AsideLayout.cshtml

```html
<!--_AsideLayout.cshtml-->
@{
  Layout = "~/Views/Shared/_Layout.cshtml";
}
@section head
{
  <!-- 这里放需要在head 中额外加入的内容--> 
  <!--可以继续保留 否则head section 到此为止-->
  @RenderSection("head",required:false)
}
@section scipts{
 <!-- 这里和上面一样，继续保留 供其他调用使用--> 
  @RenderSection("scripts",required:false)
}
<div class = "aside">
  @RenderSection("sideBar", required:false)
</div>
@RenderBody()
```

这样就很完美了。