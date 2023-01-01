# CompletableFutrure

## 简介

1. Futrure 接口的一个实现类,[[day02_JUC辅助工具类#Callable <V> 接口]]中使用的 FurtrureTask 类和他实现的是同样的接口,在 Jdk1.8 以前,使用它作为异步计算.他通过使用 `isDone()` 来判断任务是否执行完毕, `get()` 来获取任务结果,但问题在于,这两个方法的调用都需要阻塞当前线程
2. 在 Jdk1.8 中,为了解决线程阻塞的问题,添加了 CompletableFurture 类,它改为采用==回调函数==的形式,解决了 FurtureTask 在获取结果时会阻塞当前线程的问题
3. CompletableFurture 提供了强大的异步扩展功能,它能将多个任务进行转换和组合,并通过回调的方式计算处理结果.可以帮助我们大幅简化异步编程的复杂度

## 创建异步对象 api

![[异步编排-创建异步对象的四个方法.png]]
runAsync()没有返回值
supplyAsync() 有返回值
使用指定的线程池,如果没有指定,使用默认的 `ForkJoinPool.commonPool()` 作为线程池

## 计算完成时的回调方法 api

![[异步编排-回调函数.png]]
- `public CompletableFuture<T> whenComplete( BiConsumer<? super T, ? super Throwable> action)` 任务执行完毕后执行 action 
- Exceptionally () 用于执行处理异常情况
- 如果方法以 Async 结尾, 则标识会开启一个新的线程来执行这个 action, 如果没有 Async 则表示使用当前相同的线程去进行执行的回调方法

## 串行和并行
- `thenApply()` `thenApplyAsync()` 表示依赖一个线程的返回值结果, 并返回当前任务的返回值
- `thenAccept()` `thenAcceptAsync()` 表示依赖一个线程的返回值, 并做消费处理, 无返回值

## 多任务组合

多任务组合的两种形式

1. AllOf: 等待所有任务完成后结束
2. AnyOf: 只要有任意一个任务完成就算结束

## 使用异步编排优化商品详情页

在 itemService (商品详情信息整合模块) 中使用使用异步调用, 最后将他们编排为一个组, 全部执行完毕后返回数据

ItemServiceImpl 代码: 其中 executor 为自定线程池, 参见 [[day03_线程池与volatile#自定义线程池]]
```java
package com.atguigu.gmall.item.service.impl;

import com.alibaba.fastjson.JSON;
import com.atguigu.gmall.item.service.ItemService;
import com.atguigu.gmall.model.product.*;
import com.atguigu.gmall.product.client.ProductFeignClient;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;

import javax.annotation.Resource;
import java.math.BigDecimal;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ThreadPoolExecutor;
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

    @Autowired
    private ThreadPoolExecutor executor;


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

        // 供给型的异步处理方法
        CompletableFuture<SkuInfo> skuInfoCompletable = CompletableFuture.supplyAsync(() -> {
            // skuInfo
            SkuInfo skuInfo = productFeignClient.getSkuInfoBySkuId(skuId);
            resultMap.put("skuInfo", skuInfo);
            return skuInfo;
        }, executor);

        // 供给线程回调方法,开启新线程去执行回调的内容
        CompletableFuture<Void> viewCompletableFuture = skuInfoCompletable.thenAcceptAsync(skuInfo -> {
            // categoryView
            BaseCategoryView categoryView = productFeignClient.getCategoryView(skuInfo.getCategory3Id());
            resultMap.put("categoryView", categoryView);
        }, executor);
        CompletableFuture<Void> saleAttrListCompletableFuture = skuInfoCompletable.thenAcceptAsync(skuInfo -> {
            // spuSaleAttrList
            List<SpuSaleAttr> spuSaleAttrList = productFeignClient.getSpuSaleAttrListCheckBySku(skuId, skuInfo.getSpuId());
            resultMap.put("spuSaleAttrList", spuSaleAttrList);
        }, executor);
        CompletableFuture<Void> skuValueIdsMapCompletableFuture = skuInfoCompletable.thenAcceptAsync(skuInfo -> {
            // 每个spu对应的多组sku数据
            Map skuValueIdsMap = productFeignClient.getSkuValueIdsMap(skuInfo.getSpuId());
            String valuesSkuJson = JSON.toJSONString(skuValueIdsMap);
            resultMap.put("valuesSkuJson", valuesSkuJson);
        }, executor);
        CompletableFuture<Void> spuPosterListCompletableFuture = skuInfoCompletable.thenAcceptAsync(skuInfo -> {
            // spuPosterList
            List<SpuPoster> spuPosterList = productFeignClient.findSpuPosterBySpuId(skuInfo.getSpuId());
            resultMap.put("spuPosterList", spuPosterList);
        }, executor);


        // 无返回值异步调用
        CompletableFuture<Void> priceCompletableFuture = CompletableFuture.runAsync(() -> {
            // 价格
            BigDecimal skuPrice = productFeignClient.getSkuPriceBySkuId(skuId);
            resultMap.put("price", skuPrice);
        }, executor);
        CompletableFuture<Void> skuAttrListCompletableFuture = CompletableFuture.runAsync(() -> {
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
        }, executor);


        // 多任务组合,在应用执行前需要等待这一组的线程处理完成
        CompletableFuture.allOf(
                saleAttrListCompletableFuture,
                skuAttrListCompletableFuture,
                priceCompletableFuture,
                viewCompletableFuture,
                skuInfoCompletable,
                skuValueIdsMapCompletableFuture,
                spuPosterListCompletableFuture
        ).join();

        return resultMap;
    }
}
```

# 实现首页商品分类

1. 查询三级分分类视图
2. 使用 stream 流的 grouping by 进行分组处理

# 商城检索功能

## 索引(添加)功能分析

1. 只要数据库中的上架的商品都存入 [[ElasticSearch_核心概念]] 中
2. 下架商品就是将商品从 es 中删除

## 检索(查询)功能分析

检索功能不同的入口:

1. 分类导向
2. 搜索框精准搜索(常用)

检索时页面数据的两种不同获取时间:

1. 跳转到页面后由钩子函数调用后台接口获取数据
2. 在跳转前查询出数据后一同渲染

要查询的数据:

1. sku 默认商品图片
2. 价格
3. 商品名称
4. skuId
