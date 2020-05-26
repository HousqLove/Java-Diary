# 概览
JSP是Java Server Pages的缩写。文件必须放到/src/main/webapp下，文件以.jsp结尾。通过在HTML中嵌入java代码，实现动态输出的功能。

JSP通过包含特殊的指令来指明java代码：
- <%-- --%>: 注释
- <% %>: Java代码
- <%= %>: 表达式

JSP还内置有几个变量可以直接使用：
- out：表示HTTPServletResponse的PrintWriter
- session：表示当前HttpSession对象
- request：表示HTTPRequest对象

访问JSP页面时，直接指定完整的JSP路径即可。JSP本质上就是一个Servlet，无需配置映射路径，Web Server找到对应的.jsp文件后，会自动编译成Servlet再执行。

JSP的指令非常复杂：
- <%@ page import="java.io.*" %>：通过page指令引入java类
- <%@ include file="header.jsp"%>：通过include指令引入另一个JSP文件

JSP还允许自定义输出的tag。需要引入正确的taglib的jar包，还需正确声明，使用起来比较复杂。不推荐使用。