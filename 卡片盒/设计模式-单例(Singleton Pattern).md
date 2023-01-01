# 概述
采取一定的方法确保一个类只有一个实例，自行实例化并向系统提供这个实例
对于一些重量级别的对象,我们并不希望它被反复的创建(浪费系统资源)时即可使用单例模式解决
# 单例模式的三要素
1. 构造器私有化(防止外部 new 出对象实例)
2. 提供指向实例的私有静态属性
3. 提供以自身实例为返回值的公共静态方法 *getInstance()* 供外部使用这个实例
# 单例模式的使用场景
1. 需要频繁创建和销毁的对象
2. 创建对象时消耗资源过多(重量级对象),但又经常用到的对象
3. 工具类对象
4. 频繁访问的数据库或文件的对象(数据源,session 工厂等)

# 懒汉式
写法:
1. 线程不安全写法(只能在单线程下使用)
2. 线程安全(同步代码,下面的代码示例,但不利于高并发)
3. 双重检查(使用 volatile 关键字修饰实例,getInstance 方法内,进行双重检查,如果实例为空,为创建实例的代码块添加 synchronized)
4. 静态内部类(使用类的加载机制和 JVM 特性保证,推荐使用)
```java
public class SingleTonWithLazy {  
      
    private SingleTonWithLazy() {} // 构造私有化  
    private static SingleTonWithLazy singleTonWithLazyInstance;  
      
    /**  
     * 懒汉式,只在外部需要调用时才创建实例  
     *  
     * @return SingleTonWithLazy实例  
     */  
    public static synchronized SingleTonWithLazy getInstance() {  
        if (singleTonWithLazyInstance == null) {  
            singleTonWithLazyInstance = new SingleTonWithLazy();  
        }  
        return singleTonWithLazyInstance;  
    }  
}
```

**双重检查写法(推荐使用)**
```Java
public class SingleTonWithLazyDoubleCheck {  
      
    /**  
     * 使用 volatile 关键字,保证只要这个实例有变化后立即刷新到主内存中  
     */  
    private static volatile SingleTonWithLazyDoubleCheck instance;  
      
    private SingleTonWithLazyDoubleCheck() {  
    }  
      
    /**  
     * 双重检查的公共静态方法  
     *  
     * @return SingleTonWithLazyDoubleCheck 实例  
     */  
    public static SingleTonWithLazyDoubleCheck getInstance() {  
        if (instance == null) {  
            // 第一次检查为空时,立即将这个类加锁,保证只有一个线程操作以下的步骤  
            synchronized (SingleTonWithLazyDoubleCheck.class) {  
                if (instance == null) {  
                    instance = new SingleTonWithLazyDoubleCheck();  
                }  
            }  
        }  
        return instance;  
    }  
}
```
# 饿汉式
两种写法:
1. 静态变量写法(下面的代码示例)
2. 静态代码块写法(将类的实例化过程放在静态代码块中)
```java
public class SingleTonWithHungry {  
      
    private SingleTonWithHungry() {}  
      
    /**  
     * 在类加载时就创建好实例对象对外提供  
     */  
    private static final SingleTonWithHungry SINGLE_TON_WITH_HUNGRY_INSTANCE = new SingleTonWithHungry();  
      
    /**  
     * 向外提供实例,没有并发问题,在高并发都使用到实例对象时,可以优先使用饿汉式  
     *  
     * @return SingleTonWithHungry实例对象  
     */  
    public static SingleTonWithHungry getSingleTonWithHungryInstance() {  
        return SINGLE_TON_WITH_HUNGRY_INSTANCE;  
    }  
}
```
- 优点: 避免了线程同步的问题,写法简单便捷
- 缺点: 他是在类装载的时候就完成实例化,如果从始至终都没有使用过这个实例就会造成内存浪费

# 单例模式 Runtime
jdk 中的 Runtime 类就是使用到了饿汉式的单例模式