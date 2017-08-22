###Server in ASP.NET-Core 

[TOC]

#### Before you read

   下面一些是来源`.NET-Core `官方文档中的一些内容，主要讲的是和`Server` 相关的一些内容，包括`Kestrel` 以及 `ANCM` . 对于理解`.NET-Core ` 中的某些方面对我比较有帮助的，整理如下。基本是英文的，不难理解，就不翻译了。



####ASP.NET Core Module



**What ASP.NET Core Module does**

​    ANCM is a native IIS module that hooks into the IIS pipeline and redirects traffic to the backend ASP.NET Core application.

![](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/servers/aspnet-core-module/_static/ancm.png)

​    Requests come in from the Web and hit the kernel mode Http.Sys driver which routes them into IIS on the primary port (80) or SSL port (443). ANCM forwards the requests to the ASP.NET Core application on the HTTP port configured for the application, which is not port 80/443.

​    Kestrel listens for traffic coming from ANCM. ANCM specifies the port via environment variable at startup, **and the `UseIISIntegration` method configures the server to listen on `http://localhost:{port}`**. There are additional checks to reject requests not from ANCM. (ANCM does not support HTTPS forwarding, so requests are forwarded over HTTP even if received by IIS over HTTPS.)

​    Kestrel picks up requests from ANCM and pushes them into the ASP.NET Core middleware pipeline, which then handles them and passes them on as `HttpContext` instances to application logic. The application's responses are then passed back to IIS, which pushes them back out to the HTTP client that initiated the requests.

**ANCM port binding overrides other port bindings**

​    ANCM generates a dynamic port to assign to the back-end process. The `UseIISIntegration` method picks up this dynamic port and configures Kestrel to listen on `http://locahost:{dynamicPort}/`. This overrides other URL configurations, such as calls to `UseUrls` or `Kestrel's Listen API` Therefore, you don't need to call `UseUrls` or Kestrel's `Listen` API when you use ANCM. **If you do call `UseUrls` or `Listen`, Kestrel listens on the port you specify when you run the app without IIS.**



####Kestrel



**When to use Kestrel with a reverse proxy**



![](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/servers/kestrel/_static/kestrel-to-internet2.png)    



![](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/servers/kestrel/_static/kestrel-to-internet.png)



​    A scenario that requires a reverse proxy is when you have multiple applications that share the same IP and port running on a single server. That doesn't work with Kestrel directly because Kestrel doesn't support sharing the same IP and port between multiple processes. **When you configure Kestrel to listen on a port, it handles all traffic for that port regardless of host header.** A reverse proxy that can share ports must then forward to Kestrel on a unique IP and port.

PS:    (About Host-Header) For example, the host header name for the URL http://www.cnblogs.com is www.cnblogs.com. 

**UseUrls limitations**

​    You can configure endpoints by calling the `UseUrls` method or using the `urls` command-line argument or the ASPNETCORE_URLS environment variable. These methods are useful if you want your code to work with servers other than Kestrel. However, be aware of these limitations:

- You can't use SSL with these methods.
- If you use both the `Listen` method and `UseUrls`, the `Listen` endpoints override the `UseUrls`endpoints.

**URL prefixes**

​    Only HTTP URL prefixes are valid; Kestrel does not support SSL when you configure URL bindings by using `UseUrls`.

- IPv4 address with port number

  ```
  http://65.55.39.10:80/

  ```

  0.0.0.0 is a special case that binds to all IPv4 addresses.

- IPv6 address with port number

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/ 

  ```

  [::] is the IPv6 equivalent of IPv4 0.0.0.0.

- Host name with port number

  ```
  http://contoso.com:80/
  http://*:80/

  ```

  Host names, `*`, and` +`, are not special. Anything that is not a recognized IP address or `localhost` will bind to all IPv4 and IPv6 IPs. If you need to bind different host names to different ASP.NET Core applications on the same port, use `HttpSys` or a reverse proxy server such as IIS, Nginx, or Apache.

- `Localhost` name with port number or loopback IP with port number

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/

  ```

  When `localhost` is specified, Kestrel tries to bind to both IPv4 and IPv6 loopback interfaces. If the requested port is in use by another service on either loopback interface, Kestrel fails to start. If either loopback interface is unavailable for any other reason (most commonly because IPv6 is not supported), Kestrel logs a warning.

####Question Linked

* https://q.cnblogs.com/q/97628/
* https://q.cnblogs.com/q/97588/

#### Link Reference

* https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x
* https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/hosting?tabs=aspnetcore2x
* PS: [Kestrel](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/servers/kestrel) is a cross-platform HTTP server based on [libuv](https://github.com/libuv/libuv), a cross-platform asynchronous I/O library.