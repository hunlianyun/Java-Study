# Javaweb过滤器

## 一、过滤器（Filter）
> 配置了过滤器之后，在访问目标资源前，会先经过过滤器处理，之后再由过滤器根据业务逻辑判断是否需要访问目标资源

## 二、过滤器使用步骤
1. 新建一个类，实现 javax.servlet.Filter 接口，该接口有三个方法
```java
public interface Filter {
    // 初始化方法，web应用启动时执行一次该方法
    public default void init(FilterConfig filterConfig) throws ServletException {}

    // 执行过滤的方法，比如可以做表单校验之类的操作
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    // Servlet容器在销毁过滤器实例前调用该方法，在该方法中释放Servlet过滤器占用的资源
    public default void destroy() {}
}
```
涉及到的几个对象：
 - FilterConfig：可以通过它的 `getServletContext() ` 方法获取到ServletContext对象。
 - FilterChain：调用它的 `doFilter(ServletRequest request, ServletResponse response)` 方法，表示当前过滤器执行完，交给过滤器链中下个过滤器操作，如果本过滤器已经是最后一个，则直接访问目标资源。


2. 在 web.xml 文件中配置，配置方法类似 Servlet
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app>
    <filter>
        <filter-name>XxxFilter</filter-name>
        <filter-class>全类名</filter-class>
        <init-param>
            <param-name>参数名称</param-name>
            <param-value>参数值</param-value>
        </init-param>
        <dispatcher>REQUEST</dispatcher>
    </filter>
    <filter-mapping>
        <filter-name>XxxFilter</filter-name>
        <!-- 过滤路径 -->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
其中，dispatcher 标签有四个值可选：REQUEST/FORWARD/INCLUDE/ERROR
 - REQUEST：当用户直接访问页面时，才会调用该过滤器
 - FORWARD：如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用
 - INCLUDE：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用
 - ERROR：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用

 如果没做任何配置，默认是 `REQUEST`，也可以同时配置多个 dispatcher。
 另外，如果多个过滤器对同一个资源都进行了配置，其执行顺序取决于各个过滤器的 `<filter-mapping>` 标签在 xml 中的顺序，在执行过程中，若某一过滤器在 `doFilter()` 方法中未执行 `chain.doFilter()` 方法，后续过滤器便不会执行。
 
3. 也可以通过注解的方式来配置 Filter
`@WebFilter(urlPatterns = { "/xxx" })`
也可以配置各项参数，功能与 xml 配置类似， 不再赘述。

## 三、过滤器的应用之全站乱码问题
### 1，需求
处理请求的参数时，若参数带有中文，即会出现乱码。出现乱码的原因根据请求方式的不同有两种原因：
- get：浏览器发起 get 请求，中文按照 jsp 的 pageEncoding 编码将字符串转化为字节数组发送给服务器，因为是 get 方式，服务器以 ISO-8859-1 的方式将字节数组转化为该编码的字符串，在处理的时候把取得的字符串按照 ISO-8859-1 的编码还原成字节数组，最后用正确编码创建字符串，即得到传递的信息，也就有了如下经典代码：
```java
response.setContentType("text/hmtl;charset=utf-8");
String param = request.getPrameter("name");
String realParam = new String(param.getBytes("ISO-8859-1"), "UTF-8")
response.getWriter().println(realParam);
```
实际上，高版本 tomcat 严格按照 RFC 3986 规范进行访问解析，而 RFC 3986 规范定义了Url中只允许包含英文字母（a-zA-Z）、数字（0-9）、-\_.~4个特殊字符以及所有保留字符，RFC 3986 中指定了以下字符为保留字符：`! * ’ ( ) ; : @ & = + $ , / ? # [ ]` 。所以通常不建议在 get 请求中直接传递中文参数，如有需求可以客户端先用 Base64 对中文编码，服务器端再解码。

- post：浏览器发起 post 请求，获取参数时默认采用 ISO-8859-1 编码，因此获取到的中文参数乱码，post 请求的乱码问题解决非常简单，将请求编码和响应编码设置成一致并且支持中文的编码即可，代码如下：
```java
response.setContentType("text/hmtl;charset=utf-8");
request.setCharacterEncoding("UTF-8");
String param = request.getPrameter("name");
response.getWriter().println(param);
```

以上即为传统解决乱码问题的方案，这样做的一个问题就是，在每一个 Servlet 中都需要编写这段代码，代码重用率不高。在这种情况下，就可以使用 Filter，将所有请求过滤，并解决编码问题，就无需在每个 Servlet 中重复代码了。

### 2，步骤
- 以 post 为例，新建一个类实现 Filter 接口，将上述代码放到 doFilter 方法中即可，这里使用注解注册过滤器
```java
@WebFilter(urlPatterns = { "/*" })
public class CodeFilter implements Filter {

	public void destroy() {
	}

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                         throws IOException, ServletException {
		response.setContentType("text/html;charset=utf-8");
		request.setCharacterEncoding("utf-8");
		chain.doFilter(request, response);
	}

	public void init(FilterConfig fConfig) throws ServletException {
	}
}
```

- 之后所有的 Servlet 中都无需再重复写上述代码，直接获取参数即可
```java
String name = request.getParameter("name");
response.getWriter().println(name);
```
