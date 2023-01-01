# 什么是 Thymeleaf

- 服务器端 Java 模板引擎
- **主要作用是在静态页面上渲染动态数据**
- html 无法直接获取到 servlet 中的数据,因为它无法书写 Java 代码,所需要 Thymeleaf 将数据渲染到 html 中
- 它最大的特点就是能够直接在浏览器中打开并正确的显示模板页面,而无需启动整个 web 应用

# Thymeleaf 域对象

指将数据从 A 处**共享**到 B 处
- 页面域 pageContext,当前页面有效,离开当前页面失效
- **请求域** request,当前请求有效(地址栏没有变化/转发),离开当前请求失效
- **会话域** session,值浏览器环境不发生变化(关闭/清除记录等)即为当前会话
- 上下文域 servletContext | application,当前项目正常运行即有效,重启项目后失效

# 物理视图与逻辑视图

> /pages/user/login.html
> /pages/user/login_success.html
> /pages/user/regist.html
> /pages/user/regist_success.html
> ……
> 物理视图=视图前缀+逻辑视图+视图后缀

# MVC 架构

Model View Controller 它是控制层(Servlet)层开发的一种设计理念,主张将封装数据的『模型』、显示用户界面的『视图』、协调调度的『控制器』分开。

# 快速开始
## web 工程环境搭建
1. 导入 jar 包
2. 在 web.xml 中配置上下文初始化参数
   1. view-prefix 前缀
   2. view-suffix 后缀
3. 创建一个 Thymeleaf 模板类
4. 创建`xxxServlet extend ViewBaseServlet`
   1. 使用`processTemplate()`渲染数据
5. html 发送请求至`xxxServlet`
**需要使用 Thymeleaf 渲染的页面必须通过 Thymeleaf 中的 processTemplate()方法渲染后才能可以使用**

## [[SpringBoot]] 工程环境搭建
springboot 搭建 thymeleaf
1. 引入 web 启动器,集成 springboot 父工程
2. 在 application 配置文件中配置 spring.thymeleaf.xxx 前缀后缀等信息(将原 web.xml 中的配置用 spring 的配置文件代替)
3. 在配置的 classpath 下创建页面

## 基本语法
文本替换
> `th:text`
> 将表达式中的数值,替换标签内的文本,
> - 如表达式中获取到数据,直接将数据显示到标签中
> - 如未获取到数据,将空串(什么都不显示)显示到标签中

样式解析
> `th:utext` 
> 将响应的样式以 css 语法进行解析后再显示

value 替换
> `th:value`
> 将表达式中的数值替换到 value 中

路径替换
> `th:href / th:action / th:src`
> 替换路径



使用 Thymeleaf 中的链接传值
 ```html
 <a th:href="@{http://localhost:8800/targetUrl?id={placeholder}(placeholder=variable)}"></a>
```

## 多种存储方式
1. 使用 Map 作为入参,直接将数据 put() 进这个 Map 中即可从 Thymeleaf 取到
2. 使用 Model 入参,将自定的集合使用 `model.addAttribute(key,sourceMap)` 即可将自己 new 出来的集合使用一个 key 响应整个集合到 Themeleaf;它的 `model.addAllAttribute(sourceMap)` 方法则是将这个集合以 entry  的形式一对一对的响应到 Thymeleaf,这样做的好处就是再页面中可以直接取值,不需要遍历(默认是使用了一个上下文对象的 map,model 也是使用了它)

# Thymeleaf 中的域对象

## Servlet 中域
- request 在 doGet()或 doPost()中直接使用即可
- session `request.getSession()`
- Servlet `getServletContext()`

## Thymeleaf 中域
- request `${}`
- session `${session.}`
- application `${application.}`
- 如果{}里面什么都不加,就默认是 request

## Thymeleaf 中 param
- `${param}` 可以高效的获取参数
- 它与 request.getParameter()的作用一致
- 必须使用转发才能获取


# Thymeleaf 中内置对象 ${#}

\#request
- \${} 与 ${#request}
- 域对象和原生 HttpServletRequest 对象

\#session
- ${session.} 与{#session.}
- ${session.}内置对象
- ${#session.}原生 HttpSession 对象

 \#strings
- 使用#strings 可以操作字符串 `${#strings.isEmpty(nihao)}`
- \#lists
- 可以操作集合

# 分支与迭代
## 分支
if-unless
unless 相当于自带了一个 not

```html
<span th:if="${#strings.isEmpty(nihao)}">不存在</span>
<!--<span th:if="${not #strings.isEmpty(nihao)}">存在</span>-->
<span th:unless="${#strings.isEmpty(nihao)}">存在</span>
```

## 迭代
`th:each="存储数据名 : ${数据源}"`
# Thymeleaf 提取模板

th:fragment="header"

# OGNL 语言

Object-Graph Navigatoin Language

## OGNL 获取对象

将数据共享到域中是使用 OGNL 获取对象的前提,针对不同的域,使用不同的获取方法

- ${对象名}
- ${session.对象名}
- ${application.对象名}

## OGNL 获取对象中的属性
本质是调用这个属性对应的 getXXX()方法
- ${对象名.属性名}
- ${session.对象名.属性名}
- ${application.对象名.属性名}
