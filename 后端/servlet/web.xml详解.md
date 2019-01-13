# web.xml

- servlet元素告诉服务器自定义的servlet名称和servlet的全类名，servlet-mapping元素告诉服务器请求一个与url-pattern内容相匹配的URL时，应调用哪个servlet。

- jar命令只是一个把很多文件捆绑到一个文件的工具

- 编译servlet需要servlet-api，因此在eclipse中缺少tomcat运行时环境时，servlet会报错

- cookie默认情况下只发送给特定URL前缀

  如`http://localhost:8080/servlet/`生成的cookie只会发给请求`http://localhost:8080/servlet/xxx`的网页，而不会发给`http://localhost:8080/xxx`

  ```java
  //如下设置则可以让浏览器将cookie发送给创建此cookie的URL所在服务器的所有URL地址
  Cookie cookie2 = new Cookie("cookie2","cookie2");
  cookie2.setMaxAge(3600);
  cookie2.setPath("/");
  resp.addCookie(cookie2);
  ```

- 2.4版本的web.xml中web-app中元素的顺序是无所谓的，2.3版本的允许服务器拒绝运行其元素不按照此顺序出现的web应用

  1. servlet和servlet-mapping
  2. context-param
  3. filter和filter-mapping
  4. locale-encoding-mapping-list
  5. listener

- 自定义URL

  1. url-pattern的值必须以/或*.开头
  2. 匹配重复模式规则
     - **优先处理完全匹配的**
     - **目录映射优先于扩展名映射**，如果 /foo/* 和 *.html都是合法的url-pattern访问路径，那么优先请求URL `http://host/webAppPrefix/foo/bar.html`
     - **对于重复路径，越长路径越优先匹配**，如果/foo/* 和 /foo/bar/*都是合法的url-pattern访问路径，那么优先请求URL `http://host/webAppPrefix/foo/bar/baz.html`

- tomcat中的/相当于根目录，比如部署在`http://localhost:8080/servlet/`，则/就代表它

- welcome-file-list主要作用是保证可移植性

- error-page是一个幕后处理的实现技术，使用请求转发来访问，而不是重定向

### servlet

- 用注解版的servlet比xml中配置的servlet好用

  - xml中每次只能配置一个url-pattern，一个servlet对应多个url需要配置多次

    ```java
    //urlPatterns是一个String数组，一次就可配置多个url
    @WebServlet(name = "myServlet",urlPatterns = "/haha",
                initParams = {@WebInitParam(name = "a",value = "b")})
    public class MyServlet extends HttpServlet//继承HttpServlet
    ```

- servlet初始化参数用init-param进行配置，可通过`getServletConfig().getInitParameter("xx")`得到value

- 应用程序范围内的初始化参数，context-param，没有注解版，因为servlet3.0新增的是@WebServlet, @WebFilter, @WebListener

- load-on-startup元素中的值越小，服务器越优先加载编号小的servlet；若编号相同，服务器选择任意一个优先加载；若设置为负值，无法保证该servlet在服务器启动时加载

### filter

作用：拦截和修改servlet或JSP页面的输入请求和输出响应

对同一个请求的**执行顺序**：配置在web.xml中时，按照配置的先后顺序来调用filter；用注解配置时，按照**类名的字母表顺序**，而不是filterName的字母表顺序

创建过滤器的4个基本步骤：

 	1. 创建**实现Filter接口**的类
 	2. 将行为填充到doFilter方法中
 	3. 调用FilterChain对象的doFilter方法：它将调用下一个关联的过滤器
 	4. 将filter与url或servlet相关联：filter既可配置urlPatterns，也可配置servletNames

```java
public class MyFilter implements Filter {
    //可以放置一些成员数据，然后可在init方法中获取初始化参数，进行一波操作
    
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
		//FilterConfig对象提供对servletContext和filterName的访问
    }

    //若想使用HTTP协议的相关方法，将ServletRequest和ServletResponse转换为HTTP开头的即可
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        //若想禁止资源被调用，只要不调用doFilter，相反，filter可以将用户重定向到另一个页面（比如调用respones.sendRedirect方法）或自己产生一个响应
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```

修改响应：1）创建extends于javax.servlet.http.HttpServlet**Response**Wrapper的类

​		   2）在上述的类中添加提供一个PrintWriter对象和一个ServletOutputStream对象来缓存输出

​		   3）把此包装类传给doFilter，因为HttpServletResponseWrapper实现了HttpServletResponse接口

​		   4）调用FilterChain的doFilter后，通过2）中的方法获取原始输出，并且可以修改

​		   5）将修改后的输出发送给客户

过滤器与RequestDispatcher协同工作，注解版

```java
@WebFilter(filterName = "myFilter",dispatcherTypes = {DispatcherType.REQUEST})
public class MyFilter implements Filter 
```

- REQUEST类型（默认值）：过滤器只映射直接来自于客户端的请求，若用RequestDispatcher的方法（如forward,include）调用匹配于filter-mapping映射的资源，不会调用相关的过滤器
- FORWARD：过滤器映射将应用于RequestDispatcher.forward 方法所产生的请求
- INCLUDE：过滤器映射将应用于RequestDispatcher.include方法所产生的请求
- ERROR和ASYNC

转发和重定向：重定向与客户端直接调用没什么区别，**具体看下下来的文档**

### 应用程序生命周期事件监听器

基本步骤：

1. 实现合适的接口：ServletContextListener, ServletContextAttributeListener, HttpSessionListener, HttpSessionAttributeListenr, ServletRequestListener, ServletRequestAttributeListener，还有不常用的（HttpSessionActivationListenr, HttpSessionBindingListenr）
2. 实现响应感兴趣事件的方法：前6个监听器很有特点，奇数的有两个方法：创建和销毁，偶数的有三个方法：属性新增、修改、移除的动作
3. 获取这些重要的Web对象：**如Servlet上下文、变化后的Servlet上下文属性的名称（getName方法）以及属性的值（getValue方法）**，会话和请求也有这三
4. 使用这些对象：如getInitParamter, setAttribute, getAttribute
5. 声明监听器
6. 提供任何需要的初始化参数：如Servlet上下文监听器一般先读取初始化参数，以供所有的Servlet使用（即setAttribue方法）

```java
//在Servlet4.0中，变成默认方法了QAQ
public class MyServletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ServletContext servletContext = sce.getServletContext();
        servletContext.setAttribute("a","b");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {}
}
```

**servlet上下文与HttpSession的区别**：servlet上下文是web应用中所有的servlet共享的，而会话跟踪的数据存储在每个用户的HttpSession对象中

ServletContextAttributeListener，HttpSessionAttributeListenr，ServletRequestAttributeListener这三监听器获得属性名、属性值等后，通常将属性名与一个存储的名称相比较，确定是否是监控属性，然后再进行操作


