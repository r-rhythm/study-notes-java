## day06_Servlet



### Response

>  封装了服务器向浏览器做出响应时的响应报文



方法作用

* 获取响应流,响应数据
  * response.getWrite()
* 设置响应头(解决响应乱码)
  * response.setHeader("Content-type","text/html;charset=UTF-8")
* 重定向
  * response.sendRedirect()



请求转发与重定向的区别

* 请求转发地址栏时不变的,重定向地址栏改变
* 请求转发携带了request对象,而重定向没有携带,如果跳转页面需要携带数据时,因使用请求转发
* 转发可以访问WEB-INF下的资源,重定向不能
  * WEB-INF属于web应用私有目录,浏览器不能直接访问
* 转发浏览器发送一次请求,重定向浏览器发送两次请求



### Web路径问题

>  使用转发跳转路径地址栏不会变,导致使用相对路径不可靠,所以使用绝对路径

* 以**/**起始的路径,称之为绝对路径
* **/** 有两种语义
  * 若由服务器解析,**/** 代表上下文路径`Http://localhost:8080/dayxxx`
  * 由浏览器解析,**/** 则代表服务器路径,即为`Http://localhost:8080`

**/** 解析详情

> 可以书写 / 的位置 -> html | web.xml | 转发 | 重定向

* 由服务器解析
  * web.xml
  * 转发
* 浏览器解析
  * html(静态资源 css js等)
  * 重定向

**转发中直接加/就行,重定向还需要request.getContxtPath()+**

**html写base上下文路径/ title下**



### Web应用乱码

字符集及乱码

> UTF-8支持大量中文,GBK支持少量中文,其他字符集不支持中文

* 编码与解码使用的字符集不统一导致乱码现象

浏览器与服务器默认的编解码情况

> 服务器的编解码是一致的 ISO-8899-1
>
> 浏览器
>
> * 编码默认为 UTF-8
> * 解码为 GBK

解决乱码

* Get Tomcat8 不乱码
  * Tomcat7 在config -> server.xml 内添加`URLEncoding = "UTF-8"`
* Post
  * 将服务器解码字符集设置为 UTF-8
  * `request.setCharecterEncoding("UTF-8")`
* 响应乱码
  * 将服务器与浏览器的编解码都设置为UTF-8
  * `response.setContentType("text/html;charset=UTF-8")`

### 基于注解创建Servlet

@WebServlet 注解