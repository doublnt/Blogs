##WebSocket In ASP.NET-Core(一)

### What Is WebSocket?

&emsp;`WebSocket` 是一种在单个 `TCP` 连接上进行**全双工**通讯的协议，是建立在`TCP`上、且独立的协议。在`WebSocket API` 中，浏览器和服务器只需要完成一次握手，两者之间就可以进行持久性的连接，并进行双向数据传输。

&emsp; 为了建立`WebSocket` 连接，浏览器 通过 `Http 1.1` 协议的`101 StatusCode` 进行握手。

&emsp;以下是我本地的一个`WebSocket` 握手请求的例子：

`Client Request:`

```http
GET ws://localhost:62713/ws HTTP/1.1
Host: localhost:62713
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: http://localhost:62713
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.91 Safari/537.36
DNT: 1
Accept-Encoding: gzip, deflate, br
Accept-Language: en,zh-CN;q=0.8,zh;q=0.6,zh-TW;q=0.4
Sec-WebSocket-Key: aXo04R8eiNAZOIO1WJqXEQ==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

`Server Response:`

```http
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Date: Fri, 15 Sep 2017 03:10:05 GMT
Server: Kestrel
Upgrade: websocket
Sec-WebSocket-Accept: gmzB2zS5RSQhQT9LFZXZczHMjKQ=
```

一些关键字段的说明，

* `Connection` 必须设置为 `Upgrade`，表示客户端希望连接升级。
* `Upgrade` 字段必须设置 `Websocket`，表示希望升级到 `Websocket` 协议。
* `Sec-WebSocket-Key`： 是一个 `Base 64 Encode` 的值，服务端会用这些数据来构造出一个`SHA-1` 的信息摘要。之后进行 `Base-64` 编码，将结果作为 `Sec-WebSocket-Accept` 头的值，返回给客户端。

> *关键词： 持久连接，持久化协议，全双工*

### What Is This Article About?

&emsp;这篇文章你会了解到的是 WebSocket 在.NET-Core 中的一些基础实现和实践，首先先以官方给出的WebSocket入门，然后再构建WebSocket中间件，最后会用WebSocket 来实现一个**实时**聊天的小Demo。 

### Use WebSockets In ASP.NET Core

#### A Simple Explaination For Official Demo

&emsp;首先以官方给出的[WebSocket](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/sample)的例子入门，大致介绍一下。效果图大概就是下面这样的。

> ![](https://ws3.sinaimg.cn/large/006tNc79gy1fjp8wboehtj30sa0n8ju6.jpg)

就是我发什么信息，服务器就实时回复我什么信息。主要代码如下：首先要接受

```c#
// Configure function 
///Summary
// 这里主要是监听 WebSocket的请求，然后Invoke Echo 方法进行相关操作。比如，它接受到浏览器发来 WebSocket 的Close 命令了，那么在Echo 方法直接 await webSocket.CloseAsync(result.CloseStatus.Value... 相关操作
///Summary
app.Use(async (context, next) =>
            {
                if (context.Request.Path == "/ws")
                {
                    if (context.WebSockets.IsWebSocketRequest)
                    {
                        WebSocket webSocket = await context.WebSockets.AcceptWebSocketAsync();
                        await Echo(context, webSocket);
                    }
                    else
                    {
                        context.Response.StatusCode = 400;
                    }
                }
            });

//Echo function
 private async Task Echo(HttpContext context, WebSocket webSocket)
        {
            var buffer = new byte[1024 * 4];
            WebSocketReceiveResult result = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
            while (!result.CloseStatus.HasValue)
            {
                await webSocket.SendAsync(new ArraySegment<byte>(buffer, 0, result.Count), result.MessageType, result.EndOfMessage, CancellationToken.None);

                result = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
            }
            await webSocket.CloseAsync(result.CloseStatus.Value, result.CloseStatusDescription, CancellationToken.None);
        }
```

前端使用 js 来发送WebSocket 请求，让我们来看一下当我点击 Connect 时，到底发生了什么。下面用动图来演示一下。

>![](https://ws4.sinaimg.cn/large/006tNc79gy1fjp9cn307ig30in0j7177.gif)

是不是很熟悉，首先通过Http 发送101 ，转换协议，然后就进行WebSocket 通信了。因为在前面已经介绍过了WebSocket的工作原理了。

如果把Echo 方法中 Send 方法的 buffer修改，你就可以自己设定想要的回馈，

```c#
var abuffer = Encoding.ASCII.GetBytes("Hola, This is robert from cnblogs.");
 await webSocket.SendAsync(new ArraySegment<byte>(abuffer, 0, abuffer.Length), result.MessageType, result.EndOfMessage, CancellationToken.None);
                result = await webSocket.ReceiveAsync(new ArraySegment<byte>(abuffer), CancellationToken.None);
```

结果就如下图所示这样：

> ![](https://ws1.sinaimg.cn/large/006tNc79gy1fjpb7237jqj30u60583z6.jpg)

官方的例子讲解就到这里了。可以自己动手实践一下。接下来讲解如何基于上述的例子我们来构建一个在线实时聊天系统。

#### Building Real-Time WebSocket Demo

&emsp;效果图是下面这样的，

> ![](https://ws1.sinaimg.cn/large/006tNc79gy1fjpcsnvo7gg318k11ck6n.gif)

具体怎么实现的，下篇博文介绍。😄😄

### End Of Article

&emsp;如有陈述错误处，请多多指正。

###Reference

* [学习WebSocket协议——从顶层到底层的实现原理](https://github.com/abbshr/abbshr.github.io/issues/22)
* [ASP.NET Core WebSockets Source Code](https://github.com/aspnet/WebSockets/tree/dev/src/Microsoft.AspNetCore.WebSockets)

