> [!info] Spring Framework
> * 一个轻量级, 低侵入的 [[DI]] / [[IoC]] 和 [[AOP]] 容器开源框架, 它是依据 Rod Johnson 在其著作"*Expert one on one J2EE design and development*"中阐述的部分理念及原型衍生而来
> * Spring 提倡以"最少侵入"的方式来管理代码

# Spring 的核心概念
1. [[IoC]]
2. [[DI]]
3. [[AOP]]

# Bean 的生命周期
1. 通过构造器或工厂方法创建 bean 的实例
2. 为 bean 的属性设置值或者对其他 bean 的引用
3. 执行后置处理器的 `posterProcessBeforeInitialization()` 方法
4. 调用 bean 的初始化 `init()` 方法
5. 执行后置处理器的 `posterProcessAfterInitialization()` 方法
6. Bean 正常使用
7. 当容器关闭时, 销毁 bean

# Spring 的模块
1. [[SpringData]]
2. [[SpringSchedule]] 定时任务模块
3. [[SpringCache]]
4. [[SpringMVC]]
5. [[SpringCloud]]
6. [[SpringSecurity]]

# Spring 常用注解
- @Component 组件扫描
	- @Service
	- @Repository
- @ComponentScan 指定组件扫描路径
- @Transactional 开启事务
- @Scope 设置装配的对象为单例 *singleton* 或多例 *prototype*
