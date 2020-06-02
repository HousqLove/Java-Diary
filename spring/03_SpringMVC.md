# 概述
 现在服务端一般分为三层架构：
 - 表现层：web层，用来和客户端进行数据交互。一般采用mvc模型
 - 业务层：处理公司具体的业务逻辑
 - 持久层：用来操作数据库

 mvc是Model View Controller的简称，即模型、视图、控制器，每个部分各司其职。Model是数据模型，用来封装数据。View用来展示数据给用户。Controller用来接收用户请求，数据校验。整个流程的控制端。

 SpringMVC是一种基于Java实现的mvc设计模式的请求驱动类型的轻量级web框架，属于Spring Framework的后续产品。他通过一套注解，让一个简单的java类称为处理请求的控制器，而无须实现任何接口，还支持RESTful编程风格的请求。

# 启动过程
 要启动一个web应用时，服务器软件会第一步加载项目中的web.xml文件，通过其中的配置来启动项目。web.xml有多项标签，加载顺序为：context-param >> listener >> filter  >> servlet。
 
 SpringMVC的启动过程大致分为两个过程：
 - ContextLoaderListener初始化：实例化IoC容器，并将此容器注册到ServletContext中。
 - DispatcherServlet初始化：中央Servlet，总览请求，驱动流程。

Spring MVC提供了很多特殊的注解，用于处理请求和渲染视图。DispatcherServlet初始化过程中会使用这些特殊bean进行配置，也可以指定对应的bean。主要有：
 - 请求到处理器映射（HandlerMapping）：根据某些规则将进入容器的请求映射到具体的一系列处理器上。
 - 处理器适配器（HandlerAdapter）：拿到请求所对应的处理器后，负责调用处理器，是的DispatcherServlet无需关心具体细节。
 - 处理器异常解析器（HandlerExceptionResolver）：负责将捕获的异常映射到不同的试图上去，还支持更复杂的异常处理代码。
 - 视图解析器（ViewResolver）：将一个代表逻辑视图名的字符串，映射到实际的试图类型View上。
 - 地区解析器（LocalResolver&LocalContextResolver）: 负责解析客户端所在地区信息，为国际化的视图提供支持。
 - 主题解析器（ThemeResolver）：负责解析web应用中可用的主题。比如提供个性化定制的布局。
 - MultipartResolver：解析multi-part的传输请求。
 - FlashMap管理器（FlashMapManager）：能够存储并取回两次请求之间的FlashMap对象，可用于在请求之间传递数据，通常在请求重定向下使用。

web.xml文件中配置：
```xml
<!-- 配置contextConfigLocation初始化参数 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext.xml</param-value>
</context-param>
<!-- 配置ContextLoaderListerner -->
<listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener> 
<!-- servlet定义 -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
 # SpringMVC的执行流程
 1. 用户发送请求至前端控制器DispatcherServlet
 2. DispatcherServlet收到请求后调用处理器映射器HandlerMapping
 3. 处理器映射器找到具体的处理器（可根据xml配置、注解查找），生成处理器对象及处理器拦截器一并返回给DispatcherServlet
 4. DispatcherServlet调用处理器适配器HandlerAdapter
 5. HandlerAdapter经过适配调用具体的处理器（Controller）执行业务处理
 6. Controller执行完成返回ModelAndView
 7. HandlerAdapter将Controller返回的ModelAndView返回给DispatcherServlet
 8. DispatcherServlet将ModelAndView传给视图解析器ViewResolver
 9. ViewResolver解析后返回具体的View
 10. DispatcherServlet根据View进行渲染视图（将模型数据填充至视图）
 11. DispatcherServlet响应用户

 # applicationContext.xml中的标签
- context:annotation-config： 向Spring容器注册AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor这四个bean。
- context:component-scan：配置组件扫描，包含context:annotation-config配置。
- context:property-placeholder：加载单独配置的property文件的参数。
- import：引入其他Spring配置文件。

# 文件上传
```java
RequestMapping(path = "/form", method = RequestMethod.POST)
public String handleFormUpload(@RequestParam("name") String name, 
        @RequestParam("file") MultipartFile file) {
    if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
    }
    return "redirect:uploadFailure";
}
```

# 异常处理
常见异常处理：
- HandlerExceptionResolver：可以实现全局控制。接口方法的参数为抛出的异常，返回值ModelAndView可以指定异常显示页面。
- SimpleMappingExceptionResolver：Spring提供的一个默认的异常实现类。
- @ExceptionHandler：可以实现局部异常控制。在控制器内部定义，会接收控制器或其子类中@RequestMapping方法抛出的异常。在@ControllerAdvice类中定义，会处理相关控制器中抛出的异常。
- web.xml的error-page标签：简单处理异常和跳转，灵活程度不及HandlerExceptionResolver。
Spring的处理器异常解析器HandlerExceptionResolver接口的实现负责处理各类控制器执行过程中出现的异常。是DispatcherServlet中的特殊的bean，可以自定义配置。HandlerExceptionResolver与你在web应用描述符web.xml文件中能定义的异常映射（exception mapping）很相像，不过它比后者提供了更灵活的方式。比如它能提供异常被抛出时正在执行的是哪个处理器这样的信息。

在BaseController中使用@ExceptionHandler注解处理异常：
```java
@ExceptionHandler(Exception.class)
    public Object exceptionHandler(Exception ex, HttpServletResponse response, 
              HttpServletRequest request) throws IOException {
        String url = "";
        String msg = ex.getMessage();
        Object resultModel = null;
        try {
            if (ex.getClass() == HttpRequestMethodNotSupportedException.class) {
                url = "admin/common/500";
                System.out.println("--------毛有找到对应方法---------");
            } else if (ex.getClass() == ParameterException.class) {//自定义的异常
            } else if (ex.getClass() == UnauthorizedException.class) {
                url = "admin/common/unauth";
                System.out.println("--------毛有权限---------");
            }
            String header = req.getHeader("X-Requested-With");
            boolean isAjax = "XMLHttpRequest".equalsIgnoreCase(header);
            String method = req.getMethod();
            boolean isPost = "POST".equalsIgnoreCase(method);
            if (isAjax || isPost) {
                return Message.error(msg);
            } else {
                ModelAndView view = new ModelAndView(url);
                view.addObject("error", msg);
                view.addObject("class", ex.getClass());
                view.addObject("method", request.getRequestURI());
                return view;
            }
        } catch (Exception exception) {
            logger.error(exception.getMessage(), exception);
            return resultModel;
        } finally {
            logger.error(msg, ex);
            ex.printStackTrace();
        }
    }
```

在web.xml中处理异常：
```xml
<!-- 默认的错误处理页面 -->
<error-page>
    <error-code>403</error-code>
    <location>/403.html</location>
</error-page>
<error-page>
    <error-code>404</error-code>
    <location>/404.html</location>
</error-page>
<!-- 仅仅在调试的时候注视掉,在正式部署的时候不能注释 -->
<!-- 这样配置也是可以的，表示发生500错误的时候，转到500.jsp页面处理。 -->
<error-page> 
    <error-code>500</error-code> 
    <location>/500.html</location> 
</error-page> 
<!-- 这样的配置表示如果jsp页面或者servlet发生java.lang.Exception类型（当然包含子类）的异常就会转到500.jsp页面处理。 -->
<error-page> 
    <exception-type>java.lang.Exception</exception-type> 
    <location>/500.jsp</location> 
</error-page> 
<error-page> 
    <exception-type>java.lang.Throwable</exception-type> 
    <location>/500.jsp</location> 
</error-page>
<!-- 当error-code和exception-type都配置时，exception-type配置的页面优先级高及出现500错误，发生异常Exception时会跳转到500.jsp-->
```

如果resolverException返回了ModelAndView，会有限根据返回中的页面来显示。如果resolverException返回null，则展示web.xml中的error-page的500状态码配置的页面。

# 注解
## @RequestMapping
处理请求地址映射的注解。标注于类上表示类中请求映射的父路径。
- value：指定请求路径
- method：指定请求方法
- params：指定必须包含的参数值
- headers：指定必须包含的header值
- consumes：指定提交内容类型，如application/json、text/html。
- produces：指定返回的内容类型，仅当header中的Accept类中中包含该指定类型才返回。

## @RequestParam
将请求参数区数据映射到处理方法的参数上。
- value：参数名字
- required：是否必须，默认true
- defaultValue：默认值，没有参数时的值。自动将required设为false

## @PathVariable
将请求URL中的模板变量映射到功能处理方法的参数上。

## @ModelAttribute 
配置ModelAndView中Model值的注解，可以应用在方法参数或方法上。应用在参数上时，会从Model中取值对该参数赋值，并会将注解的参数对象添加到Model中。应用在方法上，会将该方法变为非处理请求的方法，但其他处理请求方法被调用时会首先调用该方法。

## @SessionAttributes
默认情况下，ModelMap中的属性作用域是request级别，如果希望多个请求共享ModelMap中的属性，可以将其属性转存到session中。和@ModelAttribute配合使用可以简化代码。需要清除@SessionAttribute时，使用SessionStatus.setComplete()，此时只清除@SessionAttribute的Session数据，不会清除HTTPSession的数据。

## @ResponseBody和@RequestBody
@Responsebody表示将方法的返回结果直接写入HTTP response body中。一般用于获取数据时使用。
@RequestBody将HTTP请求正文插入方法中，使用适合的HttpMessageConverter将请求体写入某个对象。系统提供有默认配置。可以处理application/json、application/xml格式的数据。

# 发展
当前，Web领域朝着前后端分离的方向去发展。SpringMVC作为一个实现MVC模式的框架，提供的Map、Model、ModelAndView也就不适用了。但是借助@RestController也可以实现一个前后端分离的网站。这样，SpringMVC中的V就不那么需要了。但是web开发难免会设计请求路径的解析，参数封装，过滤器，拦截器，鉴权等基础功能仍是不可缺少的。