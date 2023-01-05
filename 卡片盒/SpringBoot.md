# SpringBoot 简介

> SpringBoot 是一个快速构建 [[Spring]] 项目的脚手架,它整合封装了所有的框架,使用启动器即可快速方便的完成项目的构建,省去了大量繁琐的的 xml 配置

## 特点
- 自动配置 : 不需要再关注各个框架的整合配置, springboot 全部已经配置好了
- 起步依赖 : 我们在需要使用某个框架的时候, 直接添加这个框架的启动器依赖即可 , 不需要在关注 jar 包的冲突和整合

## 设计目的
- 用来简化 [[Spring]] 应用的初始搭建以及开发过程

# SpringBoot 常用注解
## 基本配置
- @SpringBootApplication 核心标记一个类为启动类,这个注解中包含了三个注解
	- @SpringBootConfiguration 组合了 @Configurtation 注解,实现配置文件的功能
	- **@EnableAutoConfiguration** 开启自动配置,添加了此注解后,它会去自动加载 spring.factories 配置文件中的124 个配置类 ,包括 DispatcherServlet 等 
	- @ComponentScan [[Spring]] 组件扫描
- @ComponentScan(basePackages = "com.xxx") 通过它可以自定义扫描规则,==它只会扫描到依赖的模块==

## 获取配置文件中的参数
- @Value("${server.port}") 获取配置文件中的对应参数,使用对应的类型接收
- @ConfigurationProperties(prefix="") 通过配置前缀可获得这个前缀下所有的参数,在这个注解标注的类中添加对应属性并提供 get/set 方法即可将将配置文件中的属性定义并绑定到 Java Bean 中,在需要自动注入对象的类上面添加 `@EnableConfigurationProperties(value=Properties clazz)` 即可使用配置文件中的参数

# SpringBoot 配置文件
## 加载顺序
1. 先加载 bootstrap.yml (引导文件配置)
2. 之后加载 application.yml (应用程序配置)
3. 是否加载第三个配置文件根据 application 中是否配置了 spring.profiles.active 决定
^663ab1

## 多环境切换下的配置文件优先级
- `spring.profiles.active: -dev|-test` 指定要使用的分支配置文件
- 如果 properties 和 yml 文件都存在，不存在 spring.profiles.active 设置，如果有重叠属性，默认以 properties 优先。


#  静态资源目录
> 在 SpringBoot 中有一个叫做 ResourceProperties 的类,这里面定义了默认的静态资源查找目录,只需要将静态资源放在这些目录中的任意一个路径下,[[SpringMVC]] 都会自动处理
- `classpath:/META-INF/resources/`
- `classpath:/resources/`
- `classpath:/static/`
- `classpath:/public/`


# 注意事项
> [!bug] yaml 配置注释问题
> **注释不能添加到配置后面,否则会返回一个 Map 不能转为 boolean 对象错误**
^c9aa67

> [!bug] DataSource自动配置问题
> 如果模块 pom 引入了数据库相关依赖,springboot 会在启动时自动配置连接,若没有配置链接地址则会报错.
> 在不需要使用数据库的模块启动类上方配置排除配置 @SpringBootApplication(exclude = DataSourceAutoConfiguration.class) 表示不自动配置数据源

# 自动装配原理

## 概述
SpringBoot 自动装配原理, 简单来说就是==自动管理, 按需加载==
1. 在 SpringBoot 启动类启动后, SpringBoot 会读取 `META-INF/spring.factories` 目录下的所有配置内容
2. SpringBoot 使用一定的条件对配置内容进行筛选, 例如 `@ConditionalOnClass({ RabbitTemplate.class, Channel.class })` 会检查相关的类是否存在 (是否引入依赖)
3. 只有当自动装配条件注解中的所有条件都满足, 该类才会自动装配

## SpringBoot 自动装配条件化注解
在自动配置类上有一些 `@ConditionalXxxx` 注解 , 这些注解的作用就是进行条件化选择。所谓条件化选择就是如果满足条件, 该配置类就生效, 如果不满足该配置类就不生效

-   `@ConditionalOnBean` ：当容器里有指定 Bean 的条件下
-   `@ConditionalOnMissingBean`：当容器里没有指定 Bean 的情况下
-   `@ConditionalOnSingleCandidate`：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
-   `@ConditionalOnClass`：当类路径下有指定类的条件下
-   `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
-   `@ConditionalOnProperty`：指定的属性是否有指定的值
-   `@ConditionalOnResource`：类路径是否有指定的值
-   `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
-   `@ConditionalOnJava`：基于 Java 版本作为判断条件
-   `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
-   `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
-   `@ConditionalOnWebApplication`：当前项目是 Web 项 目的条件下


## 自动装配实现
SpringBoot 的自动装配就是通过 `@EnableAutoConfiguration` 注解, 加载 `AutoConfigurationImportSelector` 类中的 `selectImports` 方法进而扫描 `META-INF/spring.factories` 文件下的自动配置类, 并将起配置到 IoC 容器的过程

~~SpringBoot 项目启动时, 通过 `SpringBootApplication` 中的 `@EnableAutoConfiguration` 注解来找到 `META-INF/spring.factories` 文件中的所有配置类, 并对其加载. 这些自动配置类都是通过 `AutoConfiguration` 结尾来命名~~

在主启动类上的 `@SpringBootApplication` 注解中有三个 SpringBoot 的核心注解:
1. `@SpringBootConfiguration`
2. `@EnableAutoConfiguration` 
3. `@ComponentScan`
其中的 `@EnableAutoConfiguration` 的作用为开启自动装配类, 这个注解又通过 `@Import(AutoConfigurationImportSelector.class)` 这个自动配置的核心注解来完成具体实现, 这个注解实现了 `AutoConfigurationImportSelector` 类, 在类中提供了自动配置的具体实现
自动配置的原理就是该类中的 `selectImports` 方法, 通过这个方法, 获得 `spring.factories` 文件下的一系列类名, 随后将这里类排除掉配置的 `elusive` 后加载到 IoC 容器中
