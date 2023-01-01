## Servlet_重点



> Servlet是将HTML中的数据提交到Java代码中的技术.它是一个可以直接被HTML直接访问的Java程序
>
> Java文件是不能被HTML访问的,所以此时就需要Servlet实现HTML访问功能
>
> 直接或间接实现javax.servlet.Servlet接口的实现类称之为Servlet



### Hello World

> 创建的动态 -> 生成war包 -> 部署并运行在Tomcat中

**创建Servlet**

1. 创建动态工程
2. 创建HTML(发送请求)
3. 创建Servlet(做出响应)
   1. 创建类实现Servlet接口并重写它的抽象方法
   2. 注册Servlet(设置当前类的URL属性) -> web.xml 里注册
      1. 注册全类名(通知服务器创建当前Servlet)`<servlet-class>` `<servlet-name别名`
      2. 设置URL属性(通过URL访问当前的Servlet)`<servlet-mapping>`

* html

  * ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>首页</title>
    </head>
    <body>
    <a href="HelloWorld">请求HelloServlet</a> 
       	<!--这里的url不加斜杠/-->
    	<!--他是web.xml内的servlet-mapping url-pattern-->
    </body>
    </html>
    ```

* Servlet

  * ```java
    public class HelloWorld implements Servlet { //servlet实现类
    
        @Override
        public void init(ServletConfig servletConfig) throws ServletException {
    
        }
    
        @Override
        public ServletConfig getServletConfig() {
            return null;
        }
    
        
        @Override
        public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    
            System.out.println("HelloWorld");
        }
    
        @Override
        public String getServletInfo() {
            return null;
        }
    
        @Override
        public void destroy() {
    
        }
    }
    ```

* web

  * ```xml
    <servlet>
    <!--    定义别名-->
        <servlet-name>HelloWorld</servlet-name>
    <!--    servlet实现类的全类名-->
        <servlet-class>com.atguigu.servlet.HelloWorld</servlet-class>
    </servlet>
    <servlet-mapping>
    <!--    上面的别名-->
        <servlet-name>HelloWorld</servlet-name>
    
    <!--    为当前的servlet设置URL-->
        <url-pattern>/HelloWorld</url-pattern>
    <!--    注意这里的URL要加/-->
    </servlet-mapping>
    ```

### Servlet工作原理(执行流程)

1. 浏览器向服务器发送请求url
2. 匹配servlet-mapping的url-pattern是否存在当前url
   1. 如果不存在 404结束
   2. 如果存在则通过rul最终匹配到对应的servlet-class并执行当前servlet类中的相应方法



### Servlet生命周期

> Servlet不是手动实例化的,是Servlet容器自动实例化的
>
> Servlet容器也叫Tomcat容器或服务器

* 构造器 创建时触发
  * 执行时机:第一次访问Servlet时,服务器创建Servlet对象
  * 执行次数:在整个生命周期中执行一次(**单例**)
* init() 初始化时触发
  * 执行时机:第一次访问Servlet时,在构造器之后执行init()
  * 执行次数:在整个生命周期中执行一次
* service() Servlet处理业务(请求与响应)触发
  * 执行时机:每次接收到请求时执行
  * 执行次数:在整个生命周期中执行多次
* destroy() 销毁触发
  * 执行时机:关闭服务器时执行destroy()
  * 执行次数:在整个生命周期中执行一次

> Servlet第一次接收请求时服务器创建对象,之后调用init()方法初始化Servlet,最后调用service()处理请求,做出响应.以后再次请求Servlet,只执行service(),最终关闭服务器的时候调用destroy()销毁对象资源
>
> 设置了加载时机`<load on stratup>1</load on stratup>`会在服务器启动时被创建并初始化.这里的1是优先级,**数值越小,优先级越高**



### ServletConfig & ServletContext 

Servletconfig

> 封装Servlet配置信息(xml),每个Servlet对应唯一一个ServletConfig对象,它由服务器创建,并将参数传入init()中,直接使用

* 作用
  * getServletName() 获取Servlet名称
  * **getServletContext() 获取上下文对象**
  * getServletParameters() 获取初始化参数(配置文件中的初始参数)

**ServletContext**

* Servlet上下文对象(上下文代表当前的应用程序),每个应用程序对应唯一一个上下文对象
* 获取方式
  * ServletConfig.getServletContext()
  * request.getContext()
  * session.getContext()
* 作用 servletContext.
  * getContextPath() 获取上下文路径
  * getRealPath() 获取真实路径(基于盘符的路径,部署在项目的路径)
  * getInitParameter() 获取(上下文的)初始化参数
  * getAttribute,setAttribute,removeAttribute 获取域对象

### 最终创建Servlet方式

> 1. 最好只保留service()方法, 处理请求做出响应
> 2. 提示注册Servlet

* EndServlet 它继承 HttpServlet

  * Create New Servlet
  * 取消勾选使用注解注册Servlet(现阶段)

  

  > EndServlet -> HttpServlet -> GenericServlet --> Servlet

  

* GenericServlet作用

  * 提供获取对象方法
    * getServletConfig()
    * getServletContext()
  * 将service()抽象化

* HttpServlet作用

  * 重写了service() 类型转换为Http协议
  * 里面的doGet()和doPost()就是处理请求的代码体,提交表单的方式是GET就提交到doGet(),是POST就提交到doPost()里

### **Request & Response 对象 *****

**request**对象

> **封装了浏览器向服务器发送请求时的请求报文**,该对象由服务器创建并传入service(),最终传入doGet()或doPost()

方法作用

1. 获取请求方式 
   1. request.getMethod()

2. 获取URL&URI
   1. request.getRequestURL() 基于服务器绝对路径
   2. request.getRequestURI() 基于上下文路径

3. 获取请求头信息
   1. request.getHeader()
   2. 获取Cookie信息 request.getCookies()

4. **获取请求参数**
   1. request.getParameter()  获取单个参数
   2. request.getParameterValues() 单个参数多个数值
   3. request.getParameterMap() 多个参数多个数值 

5. 请求转发(路径或页面跳转)

   1.  获取转发器 
       1.  request.getRequestDispatcher(path)

   2.  执行转发
       1.  .forward


   特点

   * 地址栏不变

6. 域对象

7. ...

