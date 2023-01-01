# 商品详情页数据分析
## 要查询的数据
`api/product/inner` 供内部调用的接口
1. 根据三级分类 id 进行三级分类查询(创建视图,使用视图 mapper)
```SQL
# 创建视图
create view base_category_view as  
select bc3.id,  
       bc3.name as category3_name,  
       bc2.id   as category2_id,  
       bc2.name as category2_name,  
       bc1.id   as category1_id,  
       bc1.name as category1_name  
from gmall_product.base_category3 bc3  
         join gmall_product.base_category2 bc2  
              on bc3.category2_id = bc2.id  
         join gmall_product.base_category1 bc1  
              on bc2.category1_id = bc1.id;
# 使用视图查询
select *  
from gmall_product.base_category_view  
where id = ?;
```
3. 查询 skuInfo 和关联的图片 skuImages
4. 通过 SpuId & SkuId 查询销售属性和对应的销售属性值(通过 spuSaleAttrValue 的 is_check 字段判定是否选中,使用 `if(sku_id is null,0,1)` 来判断是否选中)
```sql
select spa.id,  
       spa.spu_id,  
       spa.base_sale_attr_id,  
       spa.sale_attr_name,  
       spv.id,  
       spv.sale_attr_value_name,  
       if(skv.spu_id is null, 0, 1) as is_checked # 三元运算符  
from gmall_product.spu_sale_attr spa  
         join gmall_product.spu_sale_attr_value spv  
              on spa.spu_id = spv.spu_id and spv.base_sale_attr_id = spa.base_sale_attr_id  
    # 左连接查询,只要没有关联上的数据都显示null,为null的就证明是没有选中的商品属性,给他的is_checked属性赋值0  
         left join gmall_product.sku_sale_attr_value skv on skv.sale_attr_value_id = spv.id and skv.sku_id = ?  
where spa.spu_id = ?  
order by spa.base_sale_attr_id, spv.id
```
5. 同个 spu 下的所有 sku 和销售属性的对应关系(商品切换)
```SQL
# GROUP_CONCAT 是指在分组之后,把同一个组的多条数据拼接在一起,可以指定分隔符
SELECT GROUP_CONCAT(skv.sale_attr_value_id order by spv.base_sale_attr_id separator '|') value_ids,  
       skv.sku_idFROM gmall_product.sku_sale_attr_value skv  
         JOIN gmall_product.spu_sale_attr_value spv              
         ON skv.sale_attr_value_id = spv.idWHERE skv.spu_id = #{spuId}  
GROUP BY skv.sku_id
```
7. 通过 skuId 查询商品价格(商品信息保存在 [[Redis]],价格则单独做实时响应,因为可能随时改变)
8. 根据 skuId 查询平台属性
9. 查询海报 skuPoster
## 模块分析
web => service-item 汇总数据 => service-product 查询出数据
service-item 负责与前端对接,通过[[OpenFeign|远程调用]]其他接口获得数据,将数据做处理后响应给前端

# service-item 数据整合模块
## 启动排除问题
[[SpringBoot#注意事项]],注意排除 DataSourceAutoConfiguration
## 提供商品详情数据查询Api
通过 skuId 获得商品的详情信息
在返回值类型复杂且多的情况或者不知道返回什么类型给前端合适时,可以使用 Map 进行返回

# service-client 远程调用模块

