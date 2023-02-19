# HttpServlet源码分析

## 概念

> HttpServlet类是专门为Http协议准备的
>
> 在哪个包下:  jakarta.servlet.http.HttpServlet
>
> 到目前为止我们接触了servlet规范中的哪些接口？
>
> 1. jakarta.servlet.Servlet  核心接口
> 2. jakarta.servlet.ServletCaonfig  Servlet配置信息接口
> 3. jakarta.servlet.ServletContext  Servlet上下文接口
> 4. jakarta.servlet.ServletRequest  Servlet请求接口
> 5. jakarta.servlet.ServletResponse  Servlet响应接口
> 6. jakarta.servlet.ServletExcepetion Servlet异常
> 7. jakarta.servlet.GenericServlet 标准通用的Servlet类

### 源码分析

> 回忆servlet生命周期
>
> 1. 用户第一次请求
>    - Tomcat服务器通过反射机制，调用无参数的构造方法，创建Servlet对象（web.xml文件中配置的Servlet类对应的对象）
>    - Tomcat调用init()方法完成初始化
>    - Tomcat服务器吊桶Servlet对象的servlice()方法处理请求
> 2. 用户第二次请求
>    - Tomcat服务器调用service()方法处理请求
> 3. 服务器关闭
>    -  Tomcat服务器调用Servlet对象中的destroy()方法，做销毁之前的处理工作
>    - Tomcat服务器销毁Servlet对象
>
> 源码分析
>
> ```java
> public class ServletTest1 extends javax.servlet.http.HttpServlet {
> 	//用户第一次请求ServletTest1会被创建 同时执行无参数的构造方法
>     public ServletTest1(){
>         
>     }
>     public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
>         //用户第一次请求  这个类中的带参数的init()会执行 然后执行无参的init()
>           public void init(ServletConfig config) throws ServletException {
>         this.config = config;
>         this.init();
>     }
>     public void init() throws ServletException {
>     }
>         
>     }
>     //用户第一次发送请求
>       public abstract class HttpServlet extends GenericServlet {
>           public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
>         HttpServletRequest request;
>         HttpServletResponse response;
>         try {
>             request = (HttpServletRequest)req;
>             response = (HttpServletResponse)res;
>         } catch (ClassCastException var6) {
>             throw new ServletException(lStrings.getString("http.non_http"));
>         }
> 		// 调用重载的service()方法
>         this.service(request, response);
>     }
> }
>     //也就是这个方法  用来判断请求方式
>         protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
>         String method = req.getMethod();
>         long lastModified;
>         if (method.equals("GET")) {
>             lastModified = this.getLastModified(req);
>             if (lastModified == -1L) {
>                 this.doGet(req, resp);
>             } else {
>                 long ifModifiedSince;
>                 try {
>                     ifModifiedSince = req.getDateHeader("If-Modified-Since");
>                 } catch (IllegalArgumentException var9) {
>                     ifModifiedSince = -1L;
>                 }
> 
>                 if (ifModifiedSince < lastModified / 1000L * 1000L) {
>                     this.maybeSetLastModified(resp, lastModified);
>                     this.doGet(req, resp);
>                 } else {
>                     resp.setStatus(304);
>                 }
>             }
>         } else if (method.equals("HEAD")) {
>             lastModified = this.getLastModified(req);
>             this.maybeSetLastModified(resp, lastModified);
>             this.doHead(req, resp);
>         } else if (method.equals("POST")) {
>             this.doPost(req, resp);
>         } else if (method.equals("PUT")) {
>             this.doPut(req, resp);
>         } else if (method.equals("DELETE")) {
>             this.doDelete(req, resp);
>         } else if (method.equals("OPTIONS")) {
>             this.doOptions(req, resp);
>         } else if (method.equals("TRACE")) {
>             this.doTrace(req, resp);
>         } else {
>             String errMsg = lStrings.getString("http.method_not_implemented");
>             Object[] errArgs = new Object[]{method};
>             errMsg = MessageFormat.format(errMsg, errArgs);
>             resp.sendError(501, errMsg);
>         }
> 
>     }
>     /**
>     通过以上源代码进行分析，可以发现，只要HttpServlet类中的doGet()或doPost()方法执行，必然会报405
>     怎么避免405错误
>     	后端重写doget()方法，前端一定要发送get请求
>     	后端重写dopost()方法，前段一定要发送post请求
>     */
>       }
> }
> ```
>
> 结束:
>
> 1. 编写一个Servlet类，直接继承HttpServlet
> 2. 重写doget()或dopost()方法，到底重写谁，看前端发送的请求
> 3. 将Servlet类配置到web.xml文件中
> 4. 准备前端的页面(from表单)

