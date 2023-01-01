## 1.  Maven 项目依赖中作用范围 scope

> compile 编译 测试 部署 [默认]
>
> provided 编译 测试
>
> test 测试

## 2.  Maven 项 目之间的三种关系及其特征

> 依赖
>
> 继承
>
> 聚合

## 3.  如何理解框架**framework**

> 将通用的代码模板化
>
> 框架是一个可重复使用的设计构件,它规定了应用的**体系架构**,阐明了整个程序设计过程中的**依赖关系**/**责任分配**和**控制流程**
>
> 框架封装了一个应用程序中**通用的**代码部分,使用框架能让我们更专注于实现核心的业务逻辑,避免了花费大量精力重复造轮子

## 4.  如何理解 ORM

> Object Relation Mapper 对象关系映射是用于解决表中字段与类中属性名不一致的技术,使用 ORM 可以让我们在表中字段与类中属性间建立映射关系,从而能通过操作 pojo 来影响库中的数据

## 5.  MyBatis 中#{}和${}的区别是什么

> #{} 在底层使用的 preparedStatement 预编译对象对 sql 语句进行预编译处理,这样做的好处是防止了 sql 注入的安全问题,而部分场景在使用 #{} 无法解决的时候我们会使用 ${} 来作为替补,例如在需要实现动态的表查询时,由于 #{} 在编译时是使用 ? 占位符作为参数,而表名只能使用拼接的方式,所以在特定的场景下是只能使用 ${} 的

## 6.  MyBatis 中当实体类中的属性名和表中的字段名不一样，怎么办

> 1. 开启驼峰式命名映射
> 2. 在查询 sql 时为查询的字段起别名
> 3. 使用 ResaultMap 映射,一次定义即可重复使用

## 7.  MyBatis 如何获取自动生成的(主)键值？如何完成 MySQL 的批量操作

> 在标签内使用 userGenerateKey 和 KeyProperty 这两个属性来获得自动生成的主键值
>
> 使用 \<foreach> 标签拼接 Sql 语句来完成 MySql 的批量操作

## 8.  讲述下 Mybatis 的一级、二级缓存

> 缓存的作用是减少反复的 IO 操作,提高程序性能
>
> 缓存的分类
>
> * 一级缓存,默认开启.针对同一个 SqlSession
> * 二级缓存,默认关闭.针对同一个 SqlSessionFactory
>
> 缓存的访问步骤
>
> 1. 从数据库从获取数据
>    1. 将数据存放到一级缓存
>    2. 在关闭 SqlSession 前,将一级缓存的数据存放到二级缓存
> 2. 再次访问数据
>    1. 先从二级缓存中查找
>    2. 再从一级缓存中查找
>    3. 若都没有,再从数据库中查询并重复上面缓存操作

## 9.  简述 Mybatis 的动态 SQL，列出常用的 6 个标签及作用

> 动态 SQL 指的是在 MyBatis 中,Sql 语句可以通过使用 OGNL 表达式来达到简化操作的目的,无需我们再手动拼接 Sql
>
> 标签
>
> * \<if> 用于完成简单的分支判断
> * \<where> 动态插入 where 关键字及解决多出的 AND 及 OR 关键字问题
> * \<trim> 用于自定义 where 元素的功能
> * \<set> 动态的再行首插入 set 关键字并自动处理多余的逗号
> * \<choose> 再有多个条件的情况下只选择一个条件使用,类似于 Java 中的 if-else 分支结构
> * \<sql> 抽取可重复使用的 Sql 片段,使用\<include refid>引入 sql 片段

## 10.  描述下 IoC 和依赖注入，有哪些方式

> IoC Inversion of Control 控制反转,指将对象的控制权由程序员反转至 Spring,由 Spring 控制对象的生命周期,控制反转核心思想就是由 Spring 负责对象的创建。
>
> DI Dependency Injection 依赖注入,在对象创建过程中，Spring 会自动根据依赖关系，将它依赖的对象注入到当前对象中
>
> 依赖注入的方式
>
> * 构造器注入
> * Setter 注入
> * p 名称空间注入

## 11.  解释 Spring 自动装配的各种模式

> ByType 通过类中属性的类型,在 IoC 容器内查找对应的 bean 进行装配
>
> ByName 通过类中属性名,在 IoC 容器内查找对应的 bean 进行装配
>
> @Autowired 自动装配,先 ByType,再 Byname

## 12.  说一下 Spring 中支持的 bean 作用域

> * singleton 在 IoC 容器中,这个对象的实例始终为单例模式
> * prototype 在 IoC 容器中,这个对象有多个实例
> * request 单次请求有效
> * session 单次会话有效

## 13.  解释 Spring 框架中 bean 的生命周期
> 默认情况下:
> * 通过构造器或工厂方法创建 bean 的实例*
> * 为 bean 的属性赋值或设置对其他 bean 引用
> * 执行 init()方法初始化 bean
> * bean 正常使用
> * 关闭 IoC 容器对象时销毁 bean
 添加后置处理器的情况下
> * 在 init()方法调用前执行后置处理器的 postProcessBeforeInitialization()方法
> * 在 init()方法调用后执行后置处理器的 postProcessAfterInitialization()方法

## 14.  Spring 中常用的设计模式

> * 代理模式,为目标对象创建一个代理对象,以达到在不修改目标对象的基础上对其功能进行扩展
> * 单例模式,一个应用程序只有一个实例,并自动初始化向整个系统提供这个实例
> * 工厂模式,工厂模式对类的实例化过程进行了抽象处理,能够将软件模块中的对象创建和对象使用分离,使得软件得结构更加清晰
> * 模板方发模式,由一个抽象的模板类指定做某件事的模板,具体实现交给子类完成,对于不需要子类去实现的公共的方法就可以抽象到模板中,它**封装了不变的部分,扩展了可变的部分**.并且模板方法模式做到了**行为由父类控制,子类只负责实现**,大大增加了代码的统一性和规范性

## 15.  说下对 Spring 面向切面编程(AOP)的理解

> Aspect Oriented Programming 是对 OOP 纵向继承机制的补充和完善,它主要采取了横向抽取,动态织入的方法,以解决重复代码出现在不同位置的情况.例如管理事务/日志/异常等

## 16.  说下 Spring 面向切面编程(AOP)的常用术语

> * 横切关注点
> * 切入点 Pointcut
> * 连接点 Joinpoint
> * 通知 Advice
> * 代理 proxy
> * 目标 target
> * 切面 Aspect

## 17.  请描述一下 Spring 的声明式事务管理

> Spring 中的声明式事务管理,就是使用 AOP 的思想去管理事务,先将事务管理的代码横向抽取再动态织入
>
> 使用声明式事务管理步骤
>
> 1. 在配置文件中装配事务管理器 `DataSourceTransactionManager`
> 2. 开启事务注解支持 `<tx: annotation-driven></tx: annotation-driven>`
> 3. 在需要使用事务的方法或类上使用@Transactional 注解标注

## 18.  SpringMVC 中如何向请求域 equest 中共享数据

> * 将 ModelAndView 作为方法的返回值,处理响应数据
```java
>     @RequestMapping("/testModelAndView")
>         public ModelAndView testModelAndView() {
>
>             ModelAndView mv = new ModelAndView();
>
>             //设置模型数据[默认 request 域]
>             mv.addObject("username", "liuwei");
>
>             //设置视图名称[默认转发]
>             mv.setViewName("response_success");
>             return mv;
>         }
```
>
> * 使用 Map/Model/ModelMap 作为方式入参,处理响应数据

## 19.  SpringMVC 怎么样设定重定向和转发的

> SpringMVC 中默认使用转发
>
> 若返回值是以 `redirect:` 开头, 则使用重定向不拼接视图解析器设置的前后缀, 它是浏览器行为
> `forward:` 开头为转发, 它是服务器行为

## 20.  说出 Spring/SpringMVC 常用的 6 个注解，并简单描述作用

> * @Component 装配一个普通组件
> * @Repository 装配持久化层
> * @Service 装配业务层
> * @Controller 装配控制层|表述层
> * @Autowired 自动装配
> * @RequestMapping 请求映射
> * @PathVariable 获取 URL 中的占位符参数

## 21.  SpringMVC 中如何实现 RESTful 风格的数据传输和接收

> 1. 在 web.xml 中配置 `HiddenHttpMethodFilter` 过滤器
> 2. 表单必须以 POST 的方式提交
> 3. 提交参数必须包含 `_method`
> 4. 参数数值为 `PUT&Delete` 不区分大小写
> 5. 请求映射注解必须指明映射的方法,默认为 GET

## 22.  简述 SpringMVC 中如何通过 Ajax 请求返回 JSON 数据

> 1. 添加 Jackson 依赖
> 2. 使用@ResponseBody 注解标注方法,直接将结果作为返回值,@ResponseBody 会自动将其转换为 Json 格式并响应

## 23.  SpringMVC 中拦截器和 JavaEE 中 Filter 的异同

> * 过滤器属于服务器端,拦截器属于框架
> * 过滤器适用于 Servlet,拦截器用于拦截 Controller
> * 过滤器有两次执行时机,拦截器有三次
>   * preHandle
>   * postHandle
>   * afterCompletion

## 24.  简单的谈一下 SpringMVC 的工作流程

> 1. 客户端向服务器端发送请求
> 2. 执行 DispathcerServlet,判断当前 URL 是否存在映射
> 3. 若 URL 存在执行 DispatcherServlet 的 doDispatch 方法
>    1. 初始化 `HandlerMapping/HandlerExecutionChain /HandlerAdapter` 三个对象
> 4. 执行 HandlerExecutionChain 的 applyHandle()方法,如果是 true
> 5. 执行 ha.handle()触发 controller 中相应方法
> 6. 判断 controller 中是否存在异常
>    1. 有异常执行异常处理器
>    2. 无异常执行拦截器的第二个方法 `postHandle()`
>    3. 无论是否有异常都放回 ModelAndView 对象
> 7. 执行 `processDispatcherResult()` 进入视图解析器处理阶段
> 8. 执行拦截器的第三个方法 `afterCompletion`

## 25.  简单的谈一下 SpringMVC 的核心 API

> 1. DispatcherServlet 前端控制器
> 2. HandleMapping 处理器映射器,它建立了请求路径与控制器方法之间的映射
> 3. HandleExcutionChain 执行链,它里面包含了获取控制器的方法及多个拦截器
> 4. HandleAdapter 处理器适配器,通过 ha.handle 触发 controller 中相应方法
> 5. ViewResolver 视图解析器

## 26.  说下 ContextLoaderListener 的作用

> 一个用于加载配置文件的监听器,在整合 Spring 及 SpringMVC 时,由于 SpringMVC 是需要交给 Spring 管理的,而 SpringMVC 的容器对象是交给 DispatcherServlet 管理的,在这之前就需要加载出 Spring 的容器对象,所以使用监听器加载 Spring 容器对象**????**

## 27.  VMWare 三种工作模式

> * 桥接:虚拟机在访问局域网时有独立的 IP 地址
> * NAT:虚拟机在访问局域网时使用宿主机的 IP 地址
> * host-only

## 28.  写出 Linux 根目录下的 6 个常用目录及其作用

> * bin 存放着常用的命令
> * home 存放普通用户的主目录
> * root 管理员目录
> * etc 存放系统所需的配置文件
> * opt 额外安装软件所使用的目录
> * var 存放不断变化扩充的对象

## 29.  Redis 为什么那么快

> * Redis 完全基于内存,数据都存储在内存中
> * 数据结构简单,对数据的操作也简单
> * 使用多路 I/O 复用技术

## 30.  Redis 的五大数据类型

> * string 字符串
> * list 可以重复的集合
> * set 不可重复的集合
> * hash 键值对集合\<Key,filed>
> * zset 根据分数排序的有序的 set
