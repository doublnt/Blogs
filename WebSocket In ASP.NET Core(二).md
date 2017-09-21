## WebSocket In ASP.NET Core(二)

### Introduce

&emsp; 上篇[博文](http://www.cnblogs.com/xiyin/p/7555098.html)中，介绍了`WebSocket `的基本原理，以及一个简单的`Demo`用来对其有一个大致的认识。这篇博文讲的是我们平常在网站上可能会经常遇到的——实时聊天，本文就是来讲在`.NET-Core `使用`WebSocket`来实现一个“乞丐版”的在线实时聊天Demo。

> **关键词**：*Middleware*,*Real-Time*,*WebSocket*

###Before You Read.

&emsp;这个和我们上一篇博文中`Demo` 有何不同的呢？有何共同之处呢？

&emsp;相同的点是，都是网页作为客户端，使用`JavaScript `来发送和接收请求，`.NET-Core` 服务端接收到请求，发送该请求给客户端。

不同的地方呢，就像我下面这张图这样：

> ![](https://ws4.sinaimg.cn/large/006tNc79gy1fjrdg38xtkj311s0io0wq.jpg)

一次同时有多个客户端在，所以应该很清楚，我们只要在上面例子的基础上，对当前的已存在的Socket进行轮询，发送回应即可达到我们想要的效果。

### Create WebSocket Middleware

&emsp;在上个`Demo`的例子中，我们直接在`Startup` 的`Configure` 中直接写接受`WebSocket`请求。这次我们换成Middleware形式来处理。在写Middleware之前，在之前的介绍中，我们知道，需要有多个`WebSocket`，那么肯定需要一些对`WebSocket` 的`Get/Set` 处理。我简单的写了一个下面`WebScoket Manger Class` 

```c#
//WebSocketManager.cs
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Net.WebSockets;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace WebSocketManage
{
    public class WSConnectionManager
    {
        private static ConcurrentDictionary<string, WebSocket> _socketConcurrentDictionary = new ConcurrentDictionary<string, WebSocket>();

        public void AddSocket(WebSocket socket)
        {
            _socketConcurrentDictionary.TryAdd(CreateGuid(), socket);
        }

        public async Task RemoveSocket(WebSocket socket)
        {
            _socketConcurrentDictionary.TryRemove(GetSocketId(socket), out WebSocket aSocket);

            await aSocket.CloseAsync(
                closeStatus: WebSocketCloseStatus.NormalClosure,
                statusDescription: "Close by User",
                cancellationToken: CancellationToken.None).ConfigureAwait(false);
        }

        public string GetSocketId(WebSocket socket)
        {
            return _socketConcurrentDictionary.FirstOrDefault(k => k.Value == socket).Key;
        }

        public ConcurrentDictionary<string, WebSocket> GetAll()
        {
            return _socketConcurrentDictionary;
        }

        public string CreateGuid()
        {
            return Guid.NewGuid().ToString();
        }
    }
}

```

上面主要是对` WebSocket` 进行了简单的存取操作进行了封装。下面也把`WebSocket` 的`Send` 和`Recieve` 操作进行了封装。

```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.WebSockets;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace WebSocketManage
{
    public class WSHandler
    {
        protected WSConnectionManager _wsConnectionManager;

        public WSHandler(WSConnectionManager wSConnectionManager)
        {
            _wsConnectionManager = wSConnectionManager;
        }

        public async Task SendMessageAsync(
            WebSocket socket,
            string message,
            CancellationToken cancellationToken = default(CancellationToken))
        {
            var buffer = Encoding.UTF8.GetBytes(message);
            var segment = new ArraySegment<byte>(buffer);

            await socket.SendAsync(segment, WebSocketMessageType.Text, true, cancellationToken);
        }
        public async Task<string> RecieveAsync(WebSocket webSocket, CancellationToken cancellationToken)
        {
            var buffer = new ArraySegment<byte>(new byte[1024 * 8]);
            using (var ms = new MemoryStream())
            {
                WebSocketReceiveResult result;
                do
                {
                    cancellationToken.ThrowIfCancellationRequested();

                    result = await webSocket.ReceiveAsync(buffer, cancellationToken);
                    ms.Write(buffer.Array, buffer.Offset, result.Count);
                }
                while (!result.EndOfMessage);

                ms.Seek(0, SeekOrigin.Begin);
                if (result.MessageType != WebSocketMessageType.Text)
                {
                    return null;
                }

                using (var reader = new StreamReader(ms, Encoding.UTF8))
                {
                    return await reader.ReadToEndAsync();
                }
            }
        }
    }
}

```

有了上面两个辅助类之后，接下来就可以写我们自己的`RealTimeWebSocketMiddlerware` 了，

```c#
using Microsoft.AspNetCore.Http;
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.WebSockets;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using WebSocketManage;

namespace Robert.Middleware.WebSockets
{
    public class RealTimeWSMiddleware
    {
        private readonly RequestDelegate _next;
        private WSConnectionManager _wSConnectionManager { get; set; }
        private WSHandler _wsHanlder { get; set; }

        public RealTimeWSMiddleware(
            RequestDelegate next,
            WSConnectionManager wSConnectionManager,
            WSHandler wsHandler)
        {
            _next = next;
            _wSConnectionManager = wSConnectionManager;
            _wsHanlder = wsHandler;
        }

        public async Task Invoke(HttpContext httpContext)
        {
            if (httpContext.WebSockets.IsWebSocketRequest)
            {
                var cancellationToken = httpContext.RequestAborted;
                var currentWebSocket = await httpContext.WebSockets.AcceptWebSocketAsync();
                _wSConnectionManager.AddSocket(currentWebSocket);

                while (true)
                {
                    if (cancellationToken.IsCancellationRequested) break;
                    var response = await _wsHanlder.ReceiveAsync(currentWebSocket, cancellationToken);

                    if (string.IsNullOrEmpty(response) && currentWebSocket.State != WebSocketState.Open) break;

                    foreach (var item in _wSConnectionManager.GetAll())
                    {
                        if (item.Value.State == WebSocketState.Open)
                        {
                            await _wsHanlder.SendMessageAsync(item.Value, response, cancellationToken);
                        }
                        continue;
                    }
                }

                await _wSConnectionManager.RemoveSocket(currentWebSocket);
            }
            else
            {
                await _next(httpContext);
            }
        }
    }
}

```

---

&emsp;其实到这里，核心部分已经讲完了，接下来就是页面显示，发现信息，交互的问题了。在客户端还是像上篇文章中的一样，直接使用 `JavaScript `发送`WebScoket`请求。

&emsp;下面主要演示一下效果，在上篇博文的基础上，加上了用户名。

```javascript
<script>
    $(function () {
        var protocol = location.protocol === "https:" ? "wss:" : "ws:";
        var Uri = protocol + "//" + window.location.host + "/ws";
        var socket = new WebSocket(Uri);
        socket.onopen = e => {
            console.log("socket opened", e);
        };

        socket.onclose = function (e) {
            console.log("socket closed", e);
        };

        //function to receive from server.
        socket.onmessage = function (e) {
            console.log("Message:" + e.data);
            $('#msgs').append(e.data + '<br />');
        };

        socket.onerror = function (e) {
            console.error(e.data);
        };
    });
</script>
```

当写好了页面文件后，运行站点。最终运行的效果图，界面以实用为主，不求美观。 😄😄

> ![](https://ws2.sinaimg.cn/large/006tNc79gy1fjrmoqy7neg31by166nhs.gif)



### At End

&emsp;这里演示的其实只是简简单单的信息接受操作，实际环境中肯定不会这样用的，比如对用户的身份验证，可以建一个Auth 站点，还需要对信息传输加密。还会有 1对1 的聊天等等。这里只介绍了乞丐版，在参考链接中，一个`GitHub`上的大哥封装的很好的一个`WebSocke`t 中间件，如果感兴趣的可以去`fork `下来了解了解。当然等`SingleR 2.0` 出来后，这些肯定都是可以换掉的。

&emsp;在上述的中间件辅助的两个类中，还可以对一些内容进行封装，比如开始建立连接和断开连接等，还有群发信息等操作，都是可以封装的。

&emsp;内容中如果陈述错误处，请多多指正。

### Reference

* [**websocket-manager**](https://github.com/radu-matei/websocket-manager)