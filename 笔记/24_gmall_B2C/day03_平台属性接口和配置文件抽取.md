# 编写三级分类条件查询SQL
动态sql,返回BaseAttrInfo
```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.atguigu.gmall.product.mapper.BaseAttrInfoMapper">  
  
    <!-- 对于重复的字段,手动取别名,没有重复的字段则自动映射 -->  
    <resultMap id="baseAttrInfoMap" type="com.atguigu.gmall.model.product.BaseAttrInfo" autoMapping="true">  
        <!-- 主键映射 -->  
        <id property="id" column="id"/>  
  
        <!-- 一对多映射 -->  
        <collection property="attrValueList" ofType="com.atguigu.gmall.model.product.BaseAttrValue" autoMapping="true">  
            <!-- 将在查询时起的别名映射回去 -->  
            <id property="id" column="attr_value_id"/>  
        </collection>  
    </resultMap>  
  
    <select id="selectAttrInfoList" resultMap="baseAttrInfoMap">  
        SELECT            
	        bai.id,            
	        bai.attr_name,            
	        bai.category_id,            
	        bai.category_level,            
	        bai.create_time,            
	        bai.update_time,            
	        bav.id attr_value_id,            
	        bav.attr_id,            
	        bav.value_name,            
	        bav.create_time,            
	        bav.update_time        
        FROM            
	        gmall_product.base_attr_info bai        
        JOIN            
	        gmall_product.base_attr_value bav        
        ON           
	        bai.id = bav.attr_id        
        <where>  
            <if test="category1Id != null and category1Id != 0">  
                OR (bai.category_id = #{category1Id} AND bai.category_level = 1)            </if>  
            <if test="category2Id != null and category2Id != 0">  
                OR (bai.category_id = #{category2Id} AND bai.category_level = 2)            </if>  
            <if test="category3Id != null and category3Id != 0 ">  
                OR (bai.category_id = #{category3Id} AND bai.category_level = 3)            </if>  
        </where>  
        ORDER BY bai.category_level,bai.id    </select>  
</mapper>
```

# 添加平台属性接口
点击添加时,选择了哪级分类就会被添加到哪级分类下
属性表 base_attr_info,属性值表 base_attr_value
新增与修改使用同一个
```java
/**  
 * 添加或修改平台属性信息  
 *  
 * @param attrInfo BaseAttrInfo实体类  
 */  
@Transactional(rollbackFor = Exception.class) // 开启事务,所有异常都回滚  
@Override  
public void saveOrUpdateBaseAttrInfo(BaseAttrInfo attrInfo) {  
      
    if (attrInfo.getId() == null) {  
        // id 为空,添加  
        baseAttrInfoMapper.insert(attrInfo);  
    } else {  
        // id 不为空  
        LambdaQueryWrapper<BaseAttrValue> wrapper = new LambdaQueryWrapper<>();  
        wrapper.eq(BaseAttrValue::getAttrId, attrInfo.getId());  
        baseAttrValueMapper.delete(wrapper);  
        baseAttrInfoMapper.updateById(attrInfo);  
    }  
      
    attrInfo.getAttrValueList().forEach(item -> {  
        item.setAttrId(attrInfo.getId());  
        baseAttrValueMapper.insert(item);  
    });  
}
```

# 修改平台属性接口
- 提供数据回显的接口,获取平台属性值,没有直接查询,而是做了一个对应关系的维护绑定(先查询出平台属性,再查询平台属性值,最后将属性值封装到属性中),这一步是为了防止在平台属性已经删除的情况下,平台属性值还能被查询出来的情况

- 修改平台属性接口,如果每个平台属性值依次判断是否被修改的话,代码及其冗余内存效率极低,应直接将原先关联的平台属性值删除掉,使用提交的属性值做替换即可(平台属性已经被移除,对应的属性值也应该被移除)

# 搭建网关 server-gateway,前后端对接
补充了[[跨域]]相关知识点
解决 MySQL 中文乱码
1. 进入容器内部 `docker exec mysql /bin/bash`
2. 执行 `echo "character-set-server=utf8" >> /etc/mysql/mysql.conf.d/mysqld.cnf`

# 使用 Nacos 作为配置中心
将通用的配置全部抽取到nacos中,使用 .[[SpringBoot#加载顺序|bootsrap]] 配置文件从nacos中获取配置文件
![[nacos-做配置中心使用.png]]
[[Nacos#nacos 管理配置文件]]

# SPU 商品属性业务介绍
## 销售属性
它是一组商品的集合,例如某个型号的手机,这个手机型号称为 SPU,它下面的的不同颜色/内存大小等称为 SKU

spu 数据表结构关系
第一组: SPU 关系表spu_info/spu_sale_attr/spu_sale_attr_value/spu_image/spu_poster
第二组: 销售属性关系表 base_sale_attr/spu_sale_attr/spu_sale_attr_value
![[SPU-表关系.png]]

## SPU 分页列表查询