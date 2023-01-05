# 并发容器类

## 简介
并发容器类是 Java 并发编程包 `java.util.concurrent` 提供的用于解决多线程下集合类的线程安全问题的工具类统称

## 线程安全的集合类型
- CopyOnWrite 写时复制容器
- Collections. SynchronizedXxx 线程安全的集合
- ConcurrentHashMap 线程安全的 HashMap 

*注: Vector/Collections 中的迭代器默认是不同步的, 必须由程序员手动实现*

## CopyOnWrite 写时复制容器

1. CopyOnWriteArrayList
2. CopyOnWriteSet


```java
private transient volatile Object[] array; // volatile 保证线程可见性,A线程写入数据后其他线程立即可见
/*  
* 它通过一次数组的复制来实现可变操作
* */
public boolean add(E e) {  
    final ReentrantLock lock = this.lock;  
    lock.lock();  
    try {  
        Object[] elements = getArray();  
        int len = elements.length;  
        Object[] newElements = Arrays.copyOf(elements, len + 1);  
        newElements[len] = e;  
        setArray(newElements);  
        return true;  
    } finally {  
        lock.unlock();  
    }  
}
```
在 Java 并发编程中,优先使用 CopyOnWrite 写时复制容器

小结
1. 他适用于读多写少的并发场景
2. 存在的问题时,每次新增元素都要将原来的数组再复制一份,如果原数组的元素过大可能会导致 FullGC

CopyOnWrite 并**没有 HashMap 的实现**,HashMap 可以使用 Collections.synchronizedMap 或 ConcurrentHashMap() 解决 HashMap 的多线程安全问题,ConcurrentHashMap 使用了乐观锁(CAS)的思想来解决

# JUC 辅助类

## 作用
在开发中可能会需要主线程开启多个子线程,等待子线程之后完毕再进行汇总,这就涉及到了线程间的同步与分工问题.使用 JUC 的辅助类来解决这些痛点
1. CountDownLatch 倒计数器
2. CyclicBarrier 循环栅栏
3. Semaphore 信号量

## CountDownLatch
主线程在开启多个主线程时,使用 CountDownLatch(count) 指定初始计数后,主线程调用 await() 方法暂时等待,只有在其他线程调用 countDown() 方法将 count 减到 0 后主线程自动开始工作

代码示例:
```java
package com.liuwei.juc;  
  
import java.util.Random;  
import java.util.concurrent.CountDownLatch;  
import java.util.concurrent.TimeUnit;  
  
enum Countries {  
      
    QI(0, "齐"),  
    CHU(1, "楚"),  
    YAN(2, "燕"),  
    HAN(3, "韩"),  
    ZHAO(4, "赵"),  
    WEI(5, "魏");  
      
    private final int code;  
    private final String countryName;  
      
    Countries(int code, String country) {  
        this.code = code;  
        this.countryName = country;  
    }  
      
    public static String getCountryByCode(Integer code) {  
        Countries[] countries = Countries.values();  
        for (Countries country : countries) {  
            if (code == country.code) {  
                return country.countryName;  
            }  
        }  
        return "";  
    }       
}  
  
/**  
 * @author: liu-wei  
 * @date: 2022/11/18,19:51  
 */public class CountDownLatchDemo {  
      
    public static void main(String[] args) throws InterruptedException {  
        // 开启到倒计数器,计数六次  
        CountDownLatch countDownLatch = new CountDownLatch(6);  
          
        // 开启六个线程  
        for (int i = 0; i < 6; i++) {  
            int finalI = i;  
            new Thread(() -> {  
                  
                try {  
                    // 模拟每个线程执行业务  
                    TimeUnit.SECONDS.sleep(new Random().nextInt(3));  
                    System.out.println(Thread.currentThread().getName() + Countries.getCountryByCode(finalI) + "了");  
                      
                    // 执行业务完毕后,调用 countDown 方法,倒计数-1  
                    countDownLatch.countDown();  
                      
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }, "秦灭").start();  
        }  
        // 在倒计数降到0之前,这个线程都暂时等待  
        countDownLatch.await();  
          
        // 倒计数为0,主线程开始执行  
        System.out.println("天下归一");  
    }  
}
```

## CyclicBarrier
一个可以重复利用的"屏蔽点".创建一个实例指定参与的线程个数及所有线程处理完毕后的最终操作,即可通过方法完成线程间的状态共享
1. CyclicBarrier(int parties, Runnable barrierAction) 创建一个 CyclicBarrier 实例,parties 指定相互等待的线程个数,Runable 命令在每个障碍点运行一次,该操作由最后一个进入屏蔽点的线程执行
2. CyclicBarrier(int parties) 创建一个实例对象,指定相互等待的线程数量
3. await() 表示当前线程已经达到屏障点,线程进入休眠状态直到所有线程达到屏障点后当前线程才会被唤醒
CountDownLatch 与 join 方法的区别?
CyclicBarrier 和 CountDownLatch 的区别?
1. CountDownLatch 只能使用一次, CyclicBarrier 可以使用 reset () 重置, 使用多次
2. #TODO

## Semaphore
信号量,它可以控制同时访问资源的线程个数,设置初始化信号量后,如果当前信号量已经全部被获取,则其他线程等待,只有其他线程释放了资源后再进行分配
- `public Semaphore(int permits) permits` 初始化信号量
- `acquire()` 占用资源,当一个线程调用这个方法时,要么获取资源成功信号量-1,要么等待直至由线程释放信号量或超时
- `release()` 释放资源,信号量+1,然后唤醒等待中的线程

代码示例:
```java
package com.liuwei.juc;  
  
import java.util.Random;  
import java.util.concurrent.Semaphore;  
import java.util.concurrent.TimeUnit;  
  
/**  
 * @author: liu-wei  
 * @date: 2022/11/18,20:34  
 */public class SemaphoreDemo {  
      
    // 案例：6辆车抢占3个车位  
    public static void main(String[] args) {  
        // 初始化信号量为3,对应资源量  
        Semaphore semaphore = new Semaphore(3);  
          
        // 开启多个线程,模拟争抢资源  
        for (int i = 0; i < 6; i++) {  
            new Thread(() -> {  
                try {  
                    // 调用 acquire(获得) 方法,抢占一个资源  
                    semaphore.acquire();  
                    System.out.println(Thread.currentThread().getName() + "抢到了一个车位");  
                      
                    // 模拟线程执行完业务逻辑后释放资源  
                    TimeUnit.SECONDS.sleep(new Random().nextInt(3));  
                    System.out.println(Thread.currentThread().getName() + "驶离车位");  
                    semaphore.release();  
                      
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }, i + "线程 => ").start();  
        }  
    }  
}
/*  
1线程 => 抢到了一个车位  
0线程 => 抢到了一个车位  
2线程 => 抢到了一个车位  // 在012线程释放资源前,其他线程只能等待
1线程 => 驶离车位  
2线程 => 驶离车位  
3线程 => 抢到了一个车位  // 其他线程释放资源后才能再次获取
4线程 => 抢到了一个车位  
3线程 => 驶离车位  
5线程 => 抢到了一个车位  
5线程 => 驶离车位  
0线程 => 驶离车位  
4线程 => 驶离车位  
*/
```


# Atomic 原子类

## 介绍
Atomic 原子类是 jdk1.5 提供的 `java.util.concurrent.atomic` 包下提供的一种用法简单, 性能高效, 线程安全的一种更新共享变量的方式
在某些场景下, 我们希望的只是保证共享变量的更新操作不会让程序的运行产生混乱, 便可以使用比锁更加轻量化且性能更高的原子类

## 实现原理
Atomic 使用 [[day03_线程池与volatile#CAS|CAS]] + [[day03_线程池与volatile#Volatile 关键字|volatile]] 的方式, 做到了在保证性能和线程安全的情况下对线程间共享变量的更新
1. 原子类基于 `Unsafe` 类与自旋操作实现
2. 使用 `volatile` 关键字修饰变量, 保证了变量一旦被某一线程修改, 其他线程能够立刻感知到并更新变量值
3. 使用 `CompareAndSwap` 比较并更新, 保证了不会发生多个线程同时修改一个共享变量成功的情况  [ABA 问题](https://javaguide.cn/java/concurrent/atomic-classes.html#atomic-%E5%8E%9F%E5%AD%90%E7%B1%BB%E4%BB%8B%E7%BB%8D)

## 常见的原子类
`java.util.concurrent.atomic` 包下共有 13 类原子类, 分为四种类型
1. 基本类型原子类 `AtomicInteger`   `AtomicLong`  `AtomicBoolean`
2. 数组类型原子类 `AtomicIntegerArray`  `AtomicLongArray`  `AtomicReferenceArray`
3. ....

# Callable\<V> 接口

## 简介
- Runable 接口的增强版
- 返回一个 V 类型的值,默认抛出异常
- 而 Runable 和 Thread 类都无返回值也不抛出异常,线程内出现异常只能使用 try-catch 捕获

## 注意
Thread 没有接收 Callable 的构造,通过使用 FutureTask(未来任务/异步调用) 

## Callable 开启新线程的步骤
1. 创建 FutureTask 实例,传入 Callable 的实现
2. 开启新线程 new Thread 传入 FutureTask 实例
3. 调用 FutureTask.get() 获取返回值,**在没有得到返回值前这个线程会被一直阻塞,尽量将它放到后面**

## Callable 和 Runable 有什么区别?
1. Runable 没有返回值,也不会抛出异常,只能通过 try-catch 捕获异常,不能继续往上抛,不利于异常的统一处理
2. 方法不同,Callable 的实现方法是 call(),Runable 的实现方法是 run()
3. Callable 可以返回一个泛型的返回值

# BlockingQueue 阻塞队列
