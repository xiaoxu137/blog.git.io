# response、reques对象的基本使用
## http协议
1. 请求
2. 请求头
3. 请求正文、参数
4. 请求对象servletRequest 响应对象servletResponse
## 响应数据
1. getOutputStream(): 字节流数据
2. getWriter(): 字符流数据
3. getOutputStream()和getWriter()互斥
>原因
>>当调用getOutputStream（）或getWriter（），服务器进行接收后。将数据进行处理，处理成HTTP所认可的格式处理完成之后，会检查会检查流对象是否关闭，如果没有关闭，服务器会自动关闭
## HTTPServletRequest对象:
1. 请求对象，从客户端发送请求，服务将会创建此对象
2. 有哪些请求
    1. 在浏览器地址栏，直接输入URL地址
    2. 超链接
    3. JavaScript中location.herf="url"
    4. form表单的提交
    >注意：如果使用get提交  那么所提交的内容是追加到后方，以?name=value&password=value这样的方式追加，前三种都是get方式提交，第四种如果methed以get方式提交，也是get方式，若是post方式提交，则是post方式
    >> get方式和post方式提交的区别
    >>> get方式
    >>>> 1. 长度有限
    >>>> 2. 安全性较差
    >>>> 3. 效率相较post较高 
      
    >>> post方式
    >>>> 1. 长度不限
    >>>> 2. 安全性较好
    >>>> 3. 效率相对get较差
# 重定向
## 基本介绍 
> 当我们使用重定向时，由response.sendRedirect("url或servlet"),服务器会根据新的url,重新发起一个新的请求用户第一次通过手动方式通过浏览器访问OneServelet.OneServelet工作完毕后，将Twoservlet地址写入到响应头location属性中，使Tomcat将302状态码写入到状态栏行。
## 实现命令 
```java
    response.sendRedirect();
    // 将地址写入到响应包中的响应头
```
## 缺点
> 重定向解决方案需要在浏览器与服务器之间进行多次往返，大量时间消耗在往返次数上，增加用户等待时间
# 请求转发
## 基本介绍
> 用户第一次通过手动方式要求浏览器访问servelet1，servlet1工作完毕后通过当前的请求对象向tomcat发送请求，申请调用servlet2，tomcat在接收到这个请后自动调用servlet2来完成剩余任务
## 实现命令
请求对象代替浏览器来向tomcat发送命令
```java
// 1. 通过当前请求对象生成资源文件申请报告对象
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/资源文件名");
        //注：一定要以‘/’开头
// 2. 将报告对象发送给tomcat
    requestDispatcher.forward("当前请求对象，当前响应对象");
    //也就是repose和request
```
## 优点 
    无论本次请求涉及到多少个servlet，用户只需要手动浏览器发送一次请求
    servlet之间调用发生在服务端计算机上，节省服务端与浏览器之间的往返次数，增加服务处理速度
## 特征 
### 请求次数
    在请求转发过程中，浏览器只发送一次请求
### 请求地址
> 只能向Tomcat服务器申请调用当前网站下资源文件地址
> request.getRequestDispatcher("/资源文件名")  注：不要写网站名，只能访问内部资源，无法访问外部资源，重定向没有此限制
## 请求方式
> 在请求转发的过程中，浏览器只发送了一个http请求协议包，参与本次请求的所有servlet共享一个请求协议包。因此，这些servlet所接收的请求方式，与浏览器发送的请求方式保持一致
## 数据乱码的解决
### 提交数据乱码的解决
    
```java
    //post提交：设置
    reques.setCharacterEncoding("utf-8");
    //get提交：需要转换
    String name=request.getParameter("name")
    name = new String(name.getBytes("iso-8859-1"),"utf-8");
```
```xml
<!--  还可以通过设置tomcat实现 -->
<Connector port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" URIEncoding="UTF-8" useBodyEncodingForURI="true">
```
### 响应数据乱码的解决
```java
    //设置在网络传递的编码，默认是iso-8859-1
    response.setCharacterEncoding("utf-8")
    //设置浏览器的编码
    response.setContentType("text/html;charSet=utf-8")
```

