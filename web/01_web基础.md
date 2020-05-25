# 概览
Java EE是Java Platform Enterprise Edition的缩写，即Java企业平台。Java EE最核心的架构是基于Servlet标准的Web服务器，开发者编写的应用程序是基于Servlet API并运行在Web服务器内部的。

Java EE还有一系列技术标准：
- EJB：Enterprise JavaBean，企业级JavaBean，早起常用语实现业务逻辑，现在基本被轻量级框架如spring取代。
- JAAS：Java Authentication and Authorization Service，一个标准的认证和授权服务，常用语企业内部，web程序通常使用更轻量级的自定义认证。
- JCA：JavaEE Connector Architecture，用于连接企业内部的EIS系统。
- JMS：Java Message Service，用于消息服务。
- JTA：Java Transaction API，用于分布式事务。
- JAX-WS：Java API for XML Web Service，用于构建基于xml的web服务。
- ...

基于Web的这种Browser/Server模式，称为B/S架构。Web页面是用HTML编写的，服务端升级后，客户端无需改动就可以使用新的版本。Web应用中，浏览器请求一个URL，服务端会把生成的HTML网页发送给浏览器，浏览器和服务器之间的传输协议是HTTP。HTTP是一个基于TCP协议之上的请求-响应协议。

浏览器发送的HTTP请求：
```
GET / HTTP/1.1
Host: www.sina.com.cn
User-Agent: Mozilla/5.0 xxx
Accept: */*
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8
```
第一行表示使用GET请求获取路径为/的资源，并使用HTTP/1.1协议，从第二行开始，每行都是以Header：Value的形式表示的HTTp头，比较常用的Http Header包括：
- Host: 请求域名
- User-Agent：客户端标识
- Accept：浏览器能接收的资源类型
- Accept-Language：浏览器偏好的语言，如text/*、image/*、*/*
- Accept-Encoding: 浏览器可以支持的压缩类型，如gzip，deflate，br

服务端的响应：
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 21932
Content-Encoding: gzip
Cache-Control: max-age=300

<html>...</html>
```
服务端响应的第一行总是版本号+空格+数字+空格+文本，数字标识响应码，其中2xx标识成功，3xx标识重定向，4xx表示客户端引发的错误，5xx表示服务端引发的错误。常见响应码：
- 200 OK：成功
- 301 Moved Permanently：该URL已经永久重定向
- 302 Found：该URL需要临时重定向
- 304 Not Modified：该资源没有修改，客户端可以使用本地缓存
- 400 Bad Request：客户端发送了一个错误的请求
- 401 Unauthorized：客户端因为身份未验证而不允许访问该URL
- 403 Forbidden：服务端因权限问题拒绝了客户端的请求
- 404 Not Found：客户端请求了一个不存在的资源
- 500 Internal Server Error：服务端处理时内部错误
- 503 Service Unavailable：服务端暂时无法处理请求

从第二行开始，服务端每一行均返回一个HTTP头。服务端常返回的HTTP Header包括：
- Content-Type：该响应内容的类型，如text/html, image/jpeg
- Content-Length: 该响应内容的长度（字节数）
- Content-Encoding：该响应压缩算法，如gzip
- Cache-Control：客户端应如何缓存，如max-age=300表示最多缓存300秒

HTTP请求都是由HTTP Header和HTTP Body构成，其中HTTP Header每行都以\r\n结束。如果遇到连续两个\r\n，那么后面的内容就是HTTP Body。通常浏览器获取的第一个资源是HTML网页，网页中如果包含JavaScript，css，图片，视频等其他资源，浏览器会根据资源的URL再次向服务端请求对应的资源。

关于HTTP协议的详细内容，参考[Mozilla开发者网站](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)。

HTTP目前有多个版本，1.0是早期版本，浏览器每次建立TCP连接后，只发送一个HTTP请求，然后关闭连接，耗时且浪费资源。因此HTTP1.1允许浏览器和服务端在同一个TCP连接上反复发送，接收多个HTTP请求和响应。HTTP2.0可以支持浏览器同时发出多个请求，但每个请求需要唯一标识，服务器可以不按请求的顺序返回多个响应，由浏览器把响应和请求对应起来。HTTP3.0为了进一步提高速度，将抛弃TCP协议改为UDP协议。