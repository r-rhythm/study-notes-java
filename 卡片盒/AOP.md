# AOP 概述
> [!info] Aspect Oriented Programming(*面向切面编程*)
> * 在不更改代码的情况下对功能进行扩展
> * 对 OOP 思想的一种扩展补充,也是 Spring 中最为重要的功能之一
> * 将非核心的周边功能业务横向提取,定义为切面,将它与核心业务代码分别独立的进行开发,再动态的织入在一起
> * AOP 的主要目的是将那些与业务无关,但是需要与业务一同调用并执行的周边功能(事务,日志等)封装起来,减少冗余的代码,统一维护
> * AOP 使用动态代理[[设计模式-代理模式]]实现,如果有接口,它使用 jdk 动态代理,没有接口则使用 cglib 动态代理

# AOP 相关术语
- 横切关注点
  - 非核心业务未提取到切面类之前称为横切关注点
- 切面
  - 切点 + 通知组成的整个操作 ~~将非核心业务提取到类中,这个类称为切面类~~
- 通知
  - 增强后的代码 ~~非核心业务提取到切面类之后称为通知~~
- 目标
  - 被代理的对象称为目标对象
- 动态代理 [[设计模式-代理模式]]
  - 在不改变原来代码的情况下,对原有的功能进行增强
  - 为目标对象创建代理对象
  - 代理对象名称以$开头
- 连接点
  - 可被增强的代码 ~~通知在代码中的具体位置,被通知前的位置称为连接点~~
- 切入点
  - ==实际增强==的方法称为切点 ~~被通知之后的位置称为切入点~~

# AspectJ

它是目前最流行的**AOP 框架**
AspectJ 使用步骤
导入 jar 包

```xml
    <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
                <version>5.3.1</version>
    </dependency>
```



在spring配置文件添加如下配置

```xml
    <!-- 开启AspecJ注解支持 -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

```java
    //在配置类中使用注解开启
    @EnableAspectJAutoProxy
```


# 切面类
## 实现
1. @Aspect 标识切面类
 ```java
      @Component
      @Aspect
      public class MyLogging {
       //...
      }
 ```

2. 在切面类中定义通知(需要增强的方法)
```java
      /**
           - 前置通知[计算之前,记录参数]
           */
          @Before(value = "execution(public int com.atguigu.aop.AopCalculatorImpl.add(int, int))")
          //value execution 定义了这个通知在那个方法前切入
          public void methodBefore(JoinPoint joinPoint){
              //获取方法名称
              String methodName = joinPoint.getSignature().getName();
              //获取参数
              Object[] args = joinPoint.getArgs();
              System.out.println("==>"+methodName+"方法,参数"+ Arrays.toString(args));
          }
```

## 切入点表达式
切入点表达式,即将通知动态的织入指定的位置,配置在通知后面如 `@Around("@annotation(com.atguigu.gmall.common.cache.GmallCache)")`
通配符
- [ * ]  代表任意权限修饰符及返回值类型, 代表任意包名,任意类名,任意方法名
- [ .. ]  可以代表任意参数数据类型
```java
    // @Before(value = "execution(public int com.atguigu.aop.AopCalculatorImpl.add(int, int))") 不使用通配符
       @Before(value = "execution(* com.atguigu.aop.AopCalculatorImpl.*(..))") // 使用通配符
    //为这个实现类下任意修饰符/返回值类型及方法套用此切入点表达式(任意参数类型)
```


重用切入点表达式
1. 提取重用切入表达式

 ```java
    @Pointcut("execution(* com.atguigu.aop.AopCalculatorImpl.*(..))")
    private void myAddress(){}
```

2. 再引入指定通知中
 ```java
    @Before("myAddress()")
    public void methodBefore(JoinPoint joinPoint){
     //...
     }
```

## JoinPoint 对象
连接点对象(封装了被代理方法的全部信息),==若是在 @Around 注解中,则必须使用 ProceedingJoinPoint 对象==
作用
1. 获取方法签名,方法签名=方法名(参数列表),
 ```java
 //获取方法签名,再通过方法签名获取方法名称
String methodName = joinPoint.getSignature().getName(); // Signature为方法签名,方法名+形参列表.注解也属于形参,也可以通过方法签名获得
 ```
2. 获取参数
 ```java
      Object[] args = joinPoint.getArgs(); // 获取被代理对象的参数
```
3. 在切面类中调用被代理对象的方法
```java
Object obj = joinPoint.proceed(args); 
```

## 各个通知特性
### 前置通知 @Before
在切入点之前执行,即使被代理的类中存在异常也会照常执行这个增强方法
```java
    //    @Before(value = "execution(public int com.atguigu.aop.AopCalculatorImpl.add(int, int))")
	@Before("myAddress()")
	//value execution 定义了这个通知在那个方法前切入
	public void methodBefore(JoinPoint joinPoint) {

		//获取方法名称
		String methodName = joinPoint.getSignature().getName();

		//获取参数
		Object[] args = joinPoint.getArgs();
		System.out.println("前置通知 ==> " + methodName + "方法,参数" + Arrays.toString(args));
	}
```

### 后置通知 @After
在切入点 **所有通知** 之后执行,如切入点中存在异常也会照常执行,类似于 try-catch-finally 中的 finally
要注意的是,==后置通知无法获得结果值==
```java
    @After("myAddress()")
	public void methodAfter(JoinPoint joinPoint) {
		
		String methodName = joinPoint.getSignature().getName();
		//后置通知是无法获得结果值的,要获得结果值需要使用返回通知
		System.out.println("后置通知 ==> " + methodName);
		
	}
 ```

### 返回通知 @AfterRetuening
在切入点返回结果之后执行,如切入点中存在异常,不会执行.方法中 return 执行即返回通知执行
注意: returning 的属性值名称必须与参数中的参数值名称保持一致
```java
    @AfterReturning(value = "myAddress()",returning = "rs")
	public void methodReturning(JoinPoint joinPoint,Object rs){

		String methodName = joinPoint.getSignature().getName();

		System.out.println("返回通知 ==> " + methodName + ",结果: rs = "+rs);

	}
```

### 异常通知 @AfterThrowing
在切入点出现异常之后执行
注意: throwing 中的属性值与参数中的参数值必须一致
```java
    @AfterThrowing(value = "myAddress()",throwing = "e")
	public void methodThrowing(JoinPoint joinPoint,Exception e){
	
		String methodName = joinPoint.getSignature().getName();
	
		System.out.println("异常通知 ==> " + methodName + ",异常:e = "+e);
	}
```


### 环绕通知 @Around
前四个通知的综合体
注意:
1. 环绕通知必须有返回值,且返回值是目标对象方法的返回结果
2. ==连接点参数必须使用 ProceedingJoinPoint==
3. 可以通过使用 proceed() 来调用被代理对象中的方法
 ```java
     @Around("myAddress()")
    public Object methodAround(ProceedingJoinPoint proceedingJoinPoint) {

	    String methodName = proceedingJoinPoint.getSignature().getName();

        Object[] args = proceedingJoinPoint.getArgs();

        //返回值,返回的是目标对象方法的返回结果
        Object rs = null;

        try {
                //前置通知
                System.out.println("前置通知 ==> " + methodName + "方法,参数" + Arrays.toString(args));

                //触发目标对象中对应方法
                rs = proceedingJoinPoint.proceed();

                //返回通知
                System.out.println("返回通知 ==> " + methodName + ",结果: rs = " + rs);
		 } catch (Throwable throwable) {
                throwable.printStackTrace();

                //异常通知
                System.out.println("异常通知 ==> " + methodName + ",异常: throwable = " + throwable);

		 } finally {

                //后置通知
                System.out.println("后置通知 ==> " + methodName + "方法,参数" + Arrays.toString(args));
        }
            return rs;
    }
```

[Aop参考文档](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-ataspectj-example)

### 总结
各通知的执行顺序:
	有异常: 前置通知 > 异常通知 > 后置通知
	无异常: 前置通知 > 返回通知 > 后置通知


# 综合案例(通过AOP+分布式锁实现缓存查询)
![[day08_AOP+Redisson实现缓存数据查询和布隆过滤器#通过AOP + 分布式锁实现商品缓存]]
