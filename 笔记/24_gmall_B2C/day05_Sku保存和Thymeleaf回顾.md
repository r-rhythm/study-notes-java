# 制作sku
## 回显准备
1. 根据 spuId 查询出销售属性和销售属性值,进行回显
2. **在需要操作多表查询时,使用通用 mapper 付出的性能代价极大(多条 SQL,多次 IO),使用自定 SQL 做关联查询**
3. 多表查询时,查询方法定义在**返回值类型**的 mapper 中
4. 根据 spuId 获取商品图片 spuImageList
```java
@controller
/**  
 * 获取商品销售属性值(Spu)  
 * * @param spuId 商品id  
 * @return 某商品对应的多个销售属性及销售属性值  
 */  
@GetMapping("/spuSaleAttrList/{spuId}")  
public Result getSpuSaleAttrBySpuId(@PathVariable Long spuId) {  
    List<SpuSaleAttr> spuSaleAttrList = baseManagerService.getSpuSaleAttrBySpuId(spuId);  
    return Result.ok(spuSaleAttrList);  
}

// @service
/**  
 * 获取商品销售属性值(Spu)  
 * * @param spuId 商品id  
 * @return 某商品对应的多个销售属性及销售属性值  
 */  
@Override  
public List<SpuSaleAttr> getSpuSaleAttrBySpuId(Long spuId) {  
      
    return spuSaleAttrMapper.selectSpuSaleAttrListBySpuId(spuId);  
      
}

@mapper
/**  
 * 获取商品销售属性值(Spu)  
 * * @param spuId 商品id  
 * @return 某商品对应的多个销售属性及销售属性值  
 */  
List<SpuSaleAttr> selectSpuSaleAttrListBySpuId(Long spuId);
```

```SQl
<!-- 自定义映射 -->  
<resultMap id="spuSaleAttrMap" type="com.atguigu.gmall.model.product.SpuSaleAttr" autoMapping="true">  
    <id property="id" column="id"/>  
    <collection property="spuSaleAttrValueList" ofType="com.atguigu.gmall.model.product.SpuSaleAttrValue"  
                autoMapping="true">  
        <id column="ssav_id" property="id"/>  
    </collection>  
</resultMap>  
  
<!-- 根据SpuId查询销售属性及对应的属性值 -->  
<select id="selectSpuSaleAttrListBySpuId" resultMap="spuSaleAttrMap">  
    SELECT 
    ssa.base_sale_attr_id,           
    ssa.id,           
    ssa.spu_id,           
    ssa.sale_attr_name,           
    ssav.id ssav_id,           
    ssav.sale_attr_value_name    
    FROM gmall_product.spu_sale_attr ssa             
    JOIN gmall_product.spu_sale_attr_value ssav                  
    ON ssa.base_sale_attr_id = ssav.base_sale_attr_id                      
    AND ssa.spu_id = ssav.spu_id    
    WHERE ssa.spu_id = #{spuId}
</select>
```
## saveSkuInfo 接口

## sku 分页显示接口

## sku 上下架接口

# 后台管理系统模块总结
基于三级分类进行管理

# 用户页面渲染-模板引擎
补充了 [[day07&08_Thymeleaf|Thymeleaf]] 相关知识点
[[day07&08_Thymeleaf#SpringBoot 工程环境搭建]]
使用 `th:include` 引入通用的内容,常用于头尾