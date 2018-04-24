今天突然收到一个bug，有用户在其浏览器环境中一直无法提交内容，使用的是UC浏览器。当换成Chrome时，内容能够正常提交。鉴于本地没有一直使用Firefox 以及Chrome，于是去下载了一个UC Browser来进行测试。本地提交时，控制台一直会报这个错误：

```javascript
Uncaught TypeError: this is not a Date object.
    at getTime (<anonymous>)
```

> ![](https://ws4.sinaimg.cn/large/006tNc79gy1fqnpfz9cwjj31kw06mwha.jpg)

当从错误信息里面，我有点懵，虽然知道是问题出在getTime 这个方法，然后在Visual Stuido 中全局搜索了一下这个方法，只找到了以下几处用法:

```javascript
(new Date()).getTime() 
```

当时有个想法是，难道这个方法在UC浏览器中不兼容，然而去Google 了 [getTime](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime) 发现全绿的呀：

> ![](https://ws2.sinaimg.cn/large/006tNc79gy1fqnpl6ql59j31kw0esq5b.jpg)

突然就失去了线索，然后采用排查法，

把这些代码一段一段的去掉，真相慢慢就出来了，我发现，即使把这段代码注释掉，还是会有这个错误。这时，我发现，不是这个问题了。但是能确定的是，因为不是Ajax内部请求的问题了，可能是外面的数据涉及到了`getTime`。 往代码上看，发现如下这句：

```javascript
questionAnswer.DateAdded = new Date();
```

于是，我注释了该语句，发现竟然能够成功提交了。真的是搞的一脸懵逼。

在Stack Overflow上发现可能是 JQuery 1.7 的bug,项目中使用的是 1.7 版本

* https://stackoverflow.com/questions/27928431/jquery-ajax-cannot-send-dates