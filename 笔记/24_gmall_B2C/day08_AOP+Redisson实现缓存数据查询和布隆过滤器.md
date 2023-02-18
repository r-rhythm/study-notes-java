# Redisson
## 简介
基于 Redis 实现的 Java 驻内存网格,提供了一系列分布式的 Java 对象与分布式服务,能够简化 Redis 的分布式操作,使得更加专注于代码
可以使用它快速便捷的完成分布式锁等操作
[快速入门](https://github.com/redisson/redisson#quick-start)
[官方文档](https://github.com/redisson/redisson/wiki/Table-of-Content)

## 整合 Redisson
1. 引入依赖
2. 添加配置类,初始化配置 `Config.fromYAML(new file("xxx.yaml"))`
3. 在配置类中创建Redission客户端实例,并注入 IOC 容器

## API 的使用
- 通过 `redisson.getLock("lockName")` 获取 Rlock 锁对象,它继承自 [[02_JUC辅助工具类|JUC]] 的 Lock 对象,所以对于 api 的使用也是相似的
- lock.tryLock() 尝试获取锁,没有获得则自旋,可以指定等待时间
- lock.unLock() 释放锁
- ...

## redisson 锁的特性
1. 它对 JUC 中的各种锁(ReentrantLock,ReadWriteLock等)提供了基于 [[Redis]] 的分布式锁解决方案
2. 它针对由于程序运行期间宕机从而无法正常释放锁所引发的死锁问题,提供了一个监控锁的**看门狗**,它的作用是在锁被手动释放前,不断的延长锁的有效期,并且在程序异常宕机后自动的释放锁.也就解决了[[分布式锁#使用 Redis 实现分布式锁]]中所提出的问题

# 使用分布式锁优化获取 sku 信息
在service-product中做判断,如果缓存中没有信息则查询数据库,将操作 DB 的方法抽取出来,只有缓存未命中的情况下才做调用
实现步骤:
1. 从 redis 中获取 skuInfo 数据
2. 存在则返回结束
3. 查询为 null
	1. 获取锁,用于解决 [[Redis#缓存击穿]]问题
	2. 访问 DB 查询数据存入数据库,并返回
	3. 释放锁(无论从缓存还是 DB 获取数据,都需要在业务完成后释放锁)

可以实现但是代码侵入性太强,通过 [[AOP]] + 注解的方式,可以更优雅的实现缓存功能

# 通过AOP + 分布式锁实现商品缓存
## 整体实现思路
1. 自定义一个注解,在注解中定义操作时需要使用的参数,在外部使用注解时传入
2. 定义一个切面类,在切面类中配置切点规则,例如 `@Around("@annotation(com.atguigu.gmall.common.cache.GmallCache)")` 表示在这个注解标记的地方进行功能的增强(也就是通知)
3. 在切面类中使用通知中定义业务代码来对外部的代码进行功能增强,在此次需求中需要使用到分布式锁+Redis 缓存

## 1.定义一个注解
注解本质上就是一个标记
1. 创建一个 Anotation 注解,使用元注解进行修饰,元注解的类型和作用如下:
	1. @Target: 指定该注解可以使用的位置(method,filed等)
	2. @Retention: 注解存在的生命周期(source,class,runtime)
	3. @Inherited: 用它修饰的类在被子类继承时是否继承该注解
	4. @Documented: 在生成文档时生成此注解的文档注释
5. 在注解中定义使用这个注解时需要传入的参数(例如 value = ?),通过不同的注解参数,来达到往 redis 中插入不同 key 的目的,具体规则定义参考[[Redis#Redis 使用规范]]

注解代码示例:
```java
/**  
 * 该注解作用于方法之上,被该注解标记的方法会优先从缓存中获取对应返回值的数据  
 *  
 * @author: liu-wēi  
 * @date: 2022/12/6,16:36  
 */@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface GmallCache {  
    /**  
     * redis 中的缓存前缀,例如 user:  
     */    String prefix() default "cache:";  
      
    /**  
     * redis 中的缓存后缀,例如 userId:xx:info  
     */    String suffix() default ":info";  
}
```

## 2.编写 AOP 切面类
1. 使用 [[AOP#环绕通知 @Around]],并标注要增强功能的类
2. 创建一个通用的数据对象
3. 通过连接点对象 joinPoint,获取要增强对象的参数
4. 在外部存储不能确定存储的类型时,使用 Json,它可以转换为任何对象

凡是使用 Object 作为返回值类型接收的数据,都要注意:==数据类型向下转型的前提是这个类型之前已经向上转型过,否则出现类型转换异常==.从redis中获取到的数据需要先转型为这个数据本身的类型,再使用 Object 自动类型提升并返回.通过 Json 工具,来实现类的转换,详见代码:
```java
package com.atguigu.gmall.common.cache;  
  
import com.alibaba.fastjson.JSONObject;  
import com.atguigu.gmall.common.constant.RedisConst;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.reflect.MethodSignature;  
import org.redisson.api.RLock;  
import org.redisson.api.RedissonClient;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.data.redis.core.RedisTemplate;  
import org.springframework.stereotype.Component;  
import org.springframework.util.StringUtils;  
  
import java.util.Arrays;  
import java.util.concurrent.TimeUnit;  
  
/**  
 * @author: liu-wēi  
 * @date: 2022/12/6,17:02  
 */@Component  
@Aspect  
public class GmallCacheAspect {  
      
    @Autowired  
    private RedissonClient redissonClient;  
      
    @Autowired  
    private RedisTemplate redisTemplate;  
      
    /**  
     * 使用aop+注解实现分布式锁+缓存  
     *  
     * @param joinPoint 连接点对象  
     * @return 所需的redis或DB中的数据  
     */  
    @Around("@annotation(com.atguigu.gmall.common.cache.GmallCache)") // 确定切入点(要增强功能的类)  
    public Object cacheAspectGmall(ProceedingJoinPoint joinPoint) throws Throwable {  
          
        // 创建通用的返回对象  
        Object obj;  
          
        // 获取方法参数信息  
        Object[] args = joinPoint.getArgs();  
          
        /*  
         * 获得从redis中获取数据的key,由注解中传入的前缀 + 参数 + 后缀组成.可以从方法签名中获取需要的注解信息  
         * 1.获取方法签名信息  
         * 2.从方法签名中获取注解的信息  
         * 3.从注解中获取前缀和后缀  
         * 4.前缀 + 方法参数 + 后缀 = Key         
         * */
        // 这个注解最终作用于方法(所以数据一定来源于方法),所以强转为方法的签名信息  
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();  
        // 获取注解  
        GmallCache annotation = signature.getMethod().getAnnotation(GmallCache.class);  
        // 获取前缀/后缀  
        String prefix = annotation.prefix();  
        String suffix = annotation.suffix();  
        // 拼接redis中用于获取和存储数据的key(将参数列表的数组转换为String格式)  
        String key = prefix + Arrays.asList(args).toString() + suffix;  
          
        // 尝试从缓存中获取所需要的数据  
        try {  
            obj = cacheHit(key, signature);  
              
            if (obj == null) {  
                /*  
                 * 未命中缓存,从DB中获取数据  
                 * 1.生成锁key  
                 * 2.尝试获取锁,获取成功访问数据库,没有获取到锁自旋  
                 * 3.释放锁  
                 * */                
                 
                String lockKey = prefix + RedisConst.SKULOCK_SUFFIX;  
                // 获得一个分布式锁对象  
                RLock lock = redissonClient.getLock(lockKey);  
                // 尝试获取锁  
                boolean tryLock = lock.tryLock();  
                  
                if (tryLock) {  
                    /*  
                     * 获取锁成功...操作数据库,查詢數據.将查询到的数据数据转换为Json字符串存入Redis中  
                     * 1.如果从数据库中查询为空,则向Redis中存入一个临时的空值,避免缓存穿透  
                     * 2.数据不为空,存入Redis             
                     * */                    
                    try {  
                        obj = joinPoint.proceed(args);  
                          
                        if (obj == null) {  
                              
                            /*  
                            这里不能使用Obj实例的方式存入,因为向下转型的的前提是这个对象向上转型过,  
                            如果使用Obj实例存入,在外部方法获取这个数据时会出现类型转换异常..  
                            所以使用反射创建对象实例更加符合逻辑也更为妥当  
                            obj = new Object();                           
                            */                                                        
                            Class returnType = signature.getReturnType();  
                            // 创建这个方法返回值的实例对象  
                            obj = returnType.newInstance();  
                            // 将数据转换为Json字符串存入Redis中  
                            redisTemplate.opsForValue().set(key, JSONObject.toJSONString(obj),  
                                    RedisConst.SKUKEY_TEMPORARY_TIMEOUT, TimeUnit.SECONDS);  
                            return obj;  
                        } else {  
                            // 将数据转换为Json字符串存入Redis中  
                            redisTemplate.opsForValue().set(key, JSONObject.toJSONString(obj),  
                                    RedisConst.SKUKEY_TIMEOUT, TimeUnit.SECONDS);  
                            return obj;  
                        }  
                    } finally {  
                        // 释放锁  
                        lock.unlock();  
                    }  
                      
                } else {  
                    // 没有获得锁,自旋.休眠一段时间后重新尝试获取数据  
                    Thread.sleep(300);  
                    this.cacheAspectGmall(joinPoint);  
                }  
                  
            } else {  
                // 命中缓存,直接返回缓存中的数据  
                return obj;  
            }  
        } catch (Throwable e) {  
            // 最终兜底的方法,因为分布式锁和缓存都使用到了Redis,  
            // 为防止Redis宕机后接口直接瘫痪,可以在这种情况下访问DB返回数据  
            e.printStackTrace();  
            return joinPoint.proceed(args);  
        }  
          
        return null;  
    }  
      
    /**  
     * 从缓存中获取数据  
     *  
     * @param key       redis中的key  
     * @param signature 方法签名  
     * @return 通过Json转换为返回值类型的Object  
     */    private Object cacheHit(String key, MethodSignature signature) {  
        // 尝试从Redis中获取数据  
        String jsonStr = (String) redisTemplate.opsForValue().get(key);  
          
        // 获取到的数据不为空,对数据进行转换 **这里的数据必须转换为对应返回值的类型,不能直接使用Obj**  
        if (!StringUtils.isEmpty(jsonStr)) {  
            // 获取注解所标记方法的返回值类型  
            Class returnType = signature.getReturnType();  
            // 将Json字符串通过Json工具转换为对应返回值类型的对象并返回  
            return JSONObject.parseObject(jsonStr, returnType);  
        }  
        return null;  
    }  
}
```


# 布隆过滤器
[官方文档](https://github.com/redisson/redisson/wiki/6.-%E5%88%86%E5%B8%83%E5%BC%8F%E5%AF%B9%E8%B1%A1#68-%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8bloom-filter)
使用步骤
1. 在存储数据模块的 SpringBoot 启动类中实现 CommandLineRunner 接口(它表示在springboot初始化后需要执行的一段代码逻辑)并实现它的 `run():void` 方法
2. 在启动类中注入 RedissonClient 客户端,用于初始化布隆过滤器
3. 设置布隆过滤器的预计统计元素于期望误差值 `bloomFilter.tryInit()` 
4. 在要使用布隆过滤器的位置使用 Redisson 客户端的 `redissonClient.getBloomFilter(布隆过滤器所存储数据的key)` 获取布隆过滤器实例
5. 在外部查询方法前就先从布隆过滤器中判断是否存在该数据 `BloomFilter.contains()` ,如果不存在即可以直接返回空值,从而避免了 [[Redis#缓存穿透]]
6. 对应的, 在生成有效数据时, 将数据的 key 存储到布隆过滤器中~~, 不存入直接获取就会被拦截~~
7. ~~它的数据都是基于 Redis 进行存储,所以在每次重启Redis后都会丢失原本的数据.此时如果有请求来获取数据,这个数据如果在布隆过滤器中没有则会直接返回空,导致不走数据库与缓存查询,但在生产环境中基本不会重启Redis,但也要切记这个特性~~

代码示例:
1. 在 SpringBoot 启动类中初始化布隆过滤器
```java
package com.atguigu.gmall.product;  
  
import com.atguigu.gmall.common.constant.RedisConst;  
import org.mybatis.spring.annotation.MapperScan;  
import org.redisson.api.RBloomFilter;  
import org.redisson.api.RedissonClient;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.CommandLineRunner;  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;  
import org.springframework.context.annotation.ComponentScan;  
  
/**  
 * @author: liu-wēi  
 * @date: 2022/11/28,12:09  
 */
@SpringBootApplication  
@ComponentScan(basePackages = "com.atguigu.gmall")  
@EnableDiscoveryClient  
@MapperScan(basePackages = "com.atguigu.gmall.product.mapper")  
public class ServiceProductApplication implements CommandLineRunner {  
    public static void main(String[] args) {  
        SpringApplication.run(ServiceProductApplication.class, args);  
    }  
      
    /**  
     * 用于初始化布隆过滤器  
     */  
    @Autowired  
    private RedissonClient redissonClient;  
      
    @Override  
    public void run(String... args) throws Exception {  
          
        long dataSize = 100000L;  
        double missProbability = 0.01;  
          
        // 获取布隆过滤器  
        RBloomFilter<Object> bloomFilter = redissonClient.getBloomFilter(RedisConst.SKU_BLOOM_FILTER);  
        /*  
          初始化数据:  
          dataSize:预估数据量  
          missProbability:误判率  
         */        
         bloomFilter.tryInit(dataSize, missProbability);  
          
    }  
}
```
2. 在添加数据的同时,必须将数据 key 也添加到布隆过滤器中,作为今后外部获取这个数据时布隆过滤器判断的依据
```Java
/**  
 * 保存具体商品的信息  
 *  
 * @param skuInfo 指定了销售属性及销售属性值的具体商品  
 */  
@Transactional(rollbackFor = Exception.class)  
@Override  
public void saveSkuInfo(SkuInfo skuInfo) {  
    // 存入skuInfo  
    skuInfoMapper.insert(skuInfo);  
    Long skuInfoId = skuInfo.getId();  
      
    // 在集合为null时,输出警告并返回一个初始化的List对象,防止 NPE    
    Optional.ofNullable(skuInfo.getSkuImageList())  
            .orElseGet(() -> {  
                System.err.println("SkuImageList()为空!!!");  
                return new ArrayList<>();  
            })  
            .forEach(skuImage -> {  
                skuImage.setSkuId(skuInfoId);  
                skuImageMapper.insert(skuImage);  
            });  
      
    /*  
     * 保存平台属性  
     * 判空后进行操作,orElse为防止NPE的兜底集合,如果原集合为null则使用它来.forEach  
     *     * */    
    Optional.ofNullable(skuInfo.getSkuAttrValueList()).orElse(new ArrayList<>())  
            .forEach(skuAttrValue -> {  
                skuAttrValue.setSkuId(skuInfoId);  
                skuAttrValueMapper.insert(skuAttrValue);  
            });  
      
    // 保存销售属性值  
    Optional.ofNullable(skuInfo.getSkuSaleAttrValueList()).orElse(new ArrayList<>())  
            .forEach(skuSaleAttrValue -> {  
                skuSaleAttrValue.setSkuId(skuInfoId);  
                skuSaleAttrValue.setSpuId(skuInfo.getSpuId());  
                skuSaleAttrValueMapper.insert(skuSaleAttrValue);  
            });  
      
      
    // 添加商品成功后,将他的id存入布隆过滤器中,作为今后查询访问时判断是否存在数据的依据  
    RBloomFilter<Object> bloomFilter = redissonClient.getBloomFilter(RedisConst.SKU_BLOOM_FILTER);  
    bloomFilter.add(skuInfoId);  
}
```
3. 在请求数据的前进行过滤,判断请求的数据是否存在,如果不存在则直接返回请求,不再查询
```java
package com.atguigu.gmall.item.service.impl;  
  
import com.alibaba.fastjson.JSON;  
import com.atguigu.gmall.common.constant.RedisConst;  
import com.atguigu.gmall.item.service.ItemService;  
import com.atguigu.gmall.model.product.*;  
import com.atguigu.gmall.product.client.ProductFeignClient;  
import org.redisson.api.RBloomFilter;  
import org.redisson.api.RedissonClient;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Service;  
import org.springframework.util.CollectionUtils;  
  
import javax.annotation.Resource;  
import java.math.BigDecimal;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
import java.util.stream.Collectors;  
  
/**  
 * @author: liu-wēi  
 * @date: 2022/12/4,16:12  
 */
@Service  
public class ItemServiceImpl implements ItemService {  
      
    @Resource  
    private ProductFeignClient productFeignClient;  
      
    @Autowired  
    private RedissonClient redissonClient;  
      
      
    /**  
     * 查询商品所有信息  
     *  
     * @param skuId 库存单元id  
     * @return  
     */  
    @Override  
    public Map<String, Object> getProductInfoBySkuId(Long skuId) {  
        // 封装数据  
        HashMap<String, Object> resultMap = new HashMap<>();  
          
        /*  
        * 使用布隆过滤器判断请求的key是否存在数据库中  
        * 1.通过key获取布隆过滤器  
        * 2.判断布隆过滤器中是否包含这个值  
        * 3.如果不包含则证明数据库中没有这个值,直接返回请求,不再执行  
        * */        
	    RBloomFilter<Object> bloomFilter = redissonClient.getBloomFilter(RedisConst.SKU_BLOOM_FILTER);  
        boolean isContains = bloomFilter.contains(skuId);  
        if (!isContains) {  
            return resultMap;  
        }  
          
          
        // skuInfo  
        SkuInfo skuInfo = productFeignClient.getSkuInfoBySkuId(skuId);  
        resultMap.put("skuInfo", skuInfo);  
          
        if (skuInfo != null) {  
            // categoryView  
            BaseCategoryView categoryView = productFeignClient.getCategoryView(skuInfo.getCategory3Id());  
            resultMap.put("categoryView", categoryView);  
              
            // spuSaleAttrList  
            List<SpuSaleAttr> spuSaleAttrList = productFeignClient.getSpuSaleAttrListCheckBySku(skuId, skuInfo.getSpuId());  
            resultMap.put("spuSaleAttrList", spuSaleAttrList);  
              
            // 每个spu对应的多组sku数据  
            Map skuValueIdsMap = productFeignClient.getSkuValueIdsMap(skuInfo.getSpuId());  
            String valuesSkuJson = JSON.toJSONString(skuValueIdsMap);  
            resultMap.put("valuesSkuJson", valuesSkuJson);  
              
            // spuPosterList  
            List<SpuPoster> spuPosterList = productFeignClient.findSpuPosterBySpuId(skuInfo.getSpuId());  
            resultMap.put("spuPosterList", spuPosterList);  
              
        }  
        // 价格  
        BigDecimal skuPrice = productFeignClient.getSkuPriceBySkuId(skuId);  
        resultMap.put("price", skuPrice);  
          
        // 商品的平台属性  
        List<BaseAttrInfo> skuAttrList = productFeignClient.getAttrList(skuId);  
          
        // 前端遍历spuSaleAttrList后直接使用skuAttr.attrName进行取值,所以要对他进行封装  
        if (!CollectionUtils.isEmpty(skuAttrList)) {  
            List<Map<String, String>> mapList = skuAttrList.stream().map(attrInfo -> {  
                Map<String, String> map = new HashMap<>();  
                map.put("attrName", attrInfo.getAttrName());  
                map.put("attrValue", attrInfo.getAttrValueList().get(0).getValueName());  
                return map;  
            }).collect(Collectors.toList());  
            resultMap.put("skuAttrList", mapList);  
        }  
          
        return resultMap;  
    }  
}
```