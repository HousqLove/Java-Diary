# 概述
要编写一个完善的HTTP服务端，以HTTP/1.1为例，需要包括：
- 识别正确和错误的HTTP请求
- 识别正确和错误的HTTP头
- 复用TCP链接
- 复用线程
- IO异常处理
- ...

这些基础工作需要耗费大量的时间，因此在JavaEE平台上，处理TCP连接，解析HTTP协议这些底层工作扔给现成的Web服务器去做，我们只需要把自己的应用跑在Web服务器上，JavaEE提供了Servlet API，我们使用Servlet API编写自己的Servlet来处理请求，Web服务器实现Servlet API接口，实现底层功能。

引入servlet的依赖包才能使用Servlet，scope应指定为provided，表示仅编译时使用，因为运行时的Web服务器本身已经提供了Servlet API相关的jar包。 Java Web 应用的打包类型是war，一个war文件必须由Web服务器加载。还需要在工程目录下创建web.xml描述文件。

我们编写的Web应用程序，由Web服务器负责加载，并创建Servlet实例，最后以多线程的模式来处理HTTP请求。这样的Web服务器称作Servlet容器。有如下特点：
- 必须由Servlet容器自动创建Servlet实例
- 只会给每个Servlet类创建唯一实例
- 会使用多线程执行doGet或doPost方法。
> Servlet中定义的字段会被多个线程同时访问，要注意线程安全。
>
> HTTPServletRequest和HTTPServletResponse实例是由Servlet容器传入的局部变量，不存在多线程访问的问题。
> 
> 在doGet等方法中，如果使用了ThreadLocal要及时清理。

整个工程结构如下：
```
webProject
|
|-- pom.xml
|-- src
    |
    |-- main
        |
        |-- java
        |-- resources
        |-- webapp
            |
            |-- WEB-INF
                |-- web.xml
```

一个Servlet总是继承自HTTPServlet，然后复写doGet()或doPost()方法。参数是HTTPServletRequest和HTTPServletResponse对象，分别代表HTTP请求和响应。

一个Web App可以有多个Servlet，分别映射不同的路径。一个请求到达Web Server总是由Dispatcher分发到对应的Servlet。

## HTTPServletRequest
HTTPServletRequest封装了一个HTTP请求，通过HTTPServletRequest提供的接口方法，可以拿到几乎全部请求信息，常用的方法有：
- getMethod()：返回请求方法，如GET，POST
- getRequestUrl()：返回请求路径，但不包括请求参数，如/hello
- getQueryString(): 返回请求参数，如name=Bob&a=1&b=2
- getParameter(name): 返回请求参数，GET请求从URL读取，POST请求从BODY读取
- getContentType(): 获取请求Body的类型, 如：application/x-www-form-urlencoded
- getContextPath(): 获取当前Web App挂载的路径，对于ROOT来说总是返回空字符串
- getCookies(): 返回请求携带的所有Cookie
- getHeader(name): 获取指定的header，不区分大小写
- getHeaderNames(): 返回所有Header名称
- getInputStream(): 如果请求带有HTTP Body，该方法将打开一个输入流用于读取Body
- getReader(): 和getInputStream类似，但打开的是Reader
- getRemoteAddr(): 返回客户端的IP地址
- getScheme(): 返回协议类型，如http，https

HTTPServletRequest还有setAttribute()和getAttribute(),可以给当前HTTPServletRequest对象附加多个Key-Value，相当于当做Map使用。

## HTTPServletResponse
HttpServletResponse封装了一个HTTP响应。响应时必须先发送Header，再发送Body。常用设置Header的方法有：
- setStatus(sc): 设置响应码，默认200
- setContentType(type): 设置Body的类型，如text/html
- setCharacterEncoding(charset): 设置字符编码，如utf-8
- setHeader(name, value): 设置一个Header的值
- addCookie(cookie): 给响应添加一个Cookie
- addHeader(name, value): 给响应添加一个Header，HTTP协议允许有多个相同的Header

写入响应时，可以通过getOutputStream获取写入流，或通过getWriter获取字符流，二者选其一。写入响应前，无需设置setContentLength()，因为底层服务器会根据写入的字符自动设置，如果写入的数据量很小，实际会先写入缓冲区，如果写入量很大，服务器会自动采用Chunked编码让浏览器能识别数据结束符而不需要设置Content-Length头部。写入完毕调用flush(),因为会复用TCP连接，同样也不能close()。

## Servlet多线程模型
一个Servlet类在服务器中只有一个实例，但服务器会使用多线程执行请求，如果Servlet中定义了字段，要注意多线程并发的问题。

## 重定向与转发
重定向（Redirect）是指当浏览器请求一个URL时，服务器返回一个重定向指令，告诉浏览器地址已经变了，请使用新的URL重新发送请求。重定向有两种，区别在于浏览器是否会缓存。：
- 302响应，称为临时重定向，使用sendRedirect()方法实现。
- 302响应，称为永久重定向。可以使用setStatus(301)和setHeader("Location", newURL)实现。

转发（Forward）是指内部转发。当一个Servlet处理请求时，他可以决定自己不继续处理，而是转发给另一个Servlet处理。

## Session和Cookie
因为HTTP是一个无状态协议，Web应用无法区分收到的请求是否是同一个浏览器发出的，为了跟踪用户状态，服务器可以向浏览器分配一个唯一ID，以Cookie的形式发送到浏览器，浏览器在后续的访问中总是附带此Cookie，从而达到识别用户的目的。基于唯一ID识别用户身份的机制称为Session。

Java EE的Servlet机制内建了对Session的支持。服务器识别Session的关键就是依靠一个名为JSESSIONID的Cookie。JSESSION ID是由Servlet容器自动创建的，目的是维护一个浏览器回话，和登录没有关系。

除了用Cookie机制实现Session外，还可以通过隐藏表单，URL末尾附加ID来追踪Session。只是很少用。

使用Session时，由于服务器把所有用户的Session都存储在内存中，如果遇到内存不足，就需要把部分不活动的Session序列化到磁盘上，这会大大降低服务器的运行效率，因此放入的Session对象要小。

在使用多台服务器时，使用Session会遇到一些额外的问题，如Session不同步，通常多台服务器集群使用反向代理作为网站入口。如果多台Web Server采用无状态集群，那么反向代理总是以轮询的方式转发给每台Web Server，这会造成一个用户在Web Server 1上存储的Session信息，在Web Server 2和3上并不存在，如果在Web Server 1登录后，后续请求如果被转发到Web Server 2和3，用户看到的还是未登录的状态。

要解决这个问题，一是在所以Web Server之间同步Session，这样会严重消耗带宽，且每个Web Server都存储所有用户的Session，内测使用率很低。另一个方案是采用粘滞回话（Sticky Session）机制，即反向代理在转发请求的时候，总是根据JSESSIONID的值判断，相同的JSESSIONID总是转发到固定的Web Server，但这需要反向代理支持。无论采用何种机制，都会使得Web Server的集群很难扩展，因此Session适用于中小型Web应用程序。对于大型的Web应用程序来说，通常要避免使用Session。

HttpSession本质上是通过一个名为JSESSION的Cookie来跟踪回话的。除了这个名称外，其他名称我们可以任意使用。

创建一个新Cookie时，除了名称和值以外，通常需要设置setPath("/")，浏览器根据此前缀决定是否发生Cookie。通过setMaxAge(second)设置Cookie的有效期，单位是秒。最后通过resp.addCookie()把他添加到响应。

如果访问的是https，还需要调用setSecure(true), 否则浏览器不会发送该Cookie。

