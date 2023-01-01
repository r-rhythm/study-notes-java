# 建立 mapping
分析此模块中 [[ElasticSearch_核心概念#核心概念]] 中的 mapping 结构, 在 es 中建立对应的 mapping 结构. 根据不同字段的性质给他们做不同处理 (例如商品 id 就不应该被分词等)
Es 默认每个增加的字段都会建立索引, 但在实际中并不是所有数据都需要建立索引的, 例如此模块中的图片路径 (用户不可能会通过图片路径进行搜索)
通过实体类来进索引创建的方式:
```java
/**  
 * 通过实体类创建出对于的映射结构到es中  
 * @return  
 */  
@Autowired  
private ElasticsearchRestTemplate restTemplate;
@GetMapping("/createIndex")  
public Result createIndex(){  
    restTemplate.createIndex(Goods.class);  // 实体类
    restTemplate.putMapping(Goods.class);  
    return Result.ok();  
}
@Data  
@Document(indexName = "goods" , shards = 3,replicas = 2)  
public class Goods {  
    /**  
     * 商品Id skuId  
     */    @Id  
    private Long id;  
  
    @Field(type = FieldType.Keyword, index = false)  // keyword表示这个字段不会被做分词处理
    private String defaultImg;  
  
    //  es 中能分词的字段，这个字段数据类型必须是 text！keyword 不分词！  
    @Field(type = FieldType.Text, analyzer = "ik_max_word")  
    private String title;  
  
    @Field(type = FieldType.Double)  
    private Double price;  
  
    //  @Field(type = FieldType.Date)   6.8.1  
    @Field(type = FieldType.Date,format = DateFormat.custom,pattern = "yyyy-MM-dd HH:mm:ss")  
    private Date createTime; // 新品  
  
    @Field(type = FieldType.Long)  
    private Long tmId;  
  
    @Field(type = FieldType.Keyword)  
    private String tmName;  
  
    @Field(type = FieldType.Keyword)  
    private String tmLogoUrl;  
  
    @Field(type = FieldType.Long)  
    private Long category1Id;  
  
    @Field(type = FieldType.Keyword)  
    private String category1Name;  
  
    @Field(type = FieldType.Long)  
    private Long category2Id;  
  
    @Field(type = FieldType.Keyword)  
    private String category2Name;  
  
    @Field(type = FieldType.Long)  
    private Long category3Id;  
  
    @Field(type = FieldType.Keyword)  
    private String category3Name;  
      
    /**  
     * 商品的热度！ 我们将商品被用户点查看的次数越多，则说明热度就越高！  
     */  
    @Field(type = FieldType.Long)  
    private Long hotScore = 0L;  
      
    /**  
     * 平台属性集合对象  
     * Nested 支持嵌套查询  
     */  
    @Field(type = FieldType.Nested)  
    private List<SearchAttr> attrs;  
}
```

# Nested 对象数据类型
默认情况下, es 会将对象类型的数据存储进行**扁平化处理**, 也就是他会将一个数组中的多个对象, 按照相同字段再进行组合
例如: 会将两组理应完全独立的对象 `{"first": "John","last": "Smith"},{"first": "Alice","last": "White"}` 
自动处理为 `user.first:["John","Alice"]; user.last:["Smith","White",]` 
==如果不对索引的数据类型做处理, 那么对象将会失去原有的数据关系维护==
Nested 类型是一种特殊的对象数据类型, 使用 nested 允许对象数组独立彼此的进行索引和查询, 通过它可以维护存入对象的属性关系
在创建索引库时, 声明嵌套对象类型为 `nested` 对应的, 在查询时也需要为他指定 `nested:type = 声明为nested类型的关系字段`
问题复现:
```Json
# 1.创建一个普通索引
PUT my_index/_doc/1
{
  "group": "fans",
  "user": [
    {"first": "John","last": "Smith"},
    {"first": "Alice","last": "White"}
  ]
}
# 2.查询
GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"user.first": "John"}},
        {"match": {"user.last": "White"}}
      ]
    }
  }
}
# 以上的查询结果意为要查询一个first为"John",last为"White"的用户,结合数据库中的数据来看,
# 这个查询语句预期应该是无法查询出结果的,但是却将姓名中包含这两个的用户全部查出来了,这就
# 与我们的需求不符, {"John","Smith"}这个对象在数据库中是不存在的,却查询出了数据
# 这就是由于在创建索引的时候没有选择使用nested类型而使用了默认的Object类型
```

解决方式:
1. Nested 关系的建立 (删除原索引重新建立)
```json
# 1.删除原索引
DELETE my_index

# 2.建立一个user数据类型为nested类型的索引并存入数据
PUT my_index
{
  "mappings": {
    "properties": {
      "user":{"type": "nested"}
    }
  }
}
# 存入数据
PUT my_index/_doc/1
{
  "group" : "fans",
  "user" : [ 
    {"first" : "John","last" :  "Smith"},
    {"first" : "Alice","last" :  "White"}
  ]
}
```

2. Nested 查询 (在查询 nested 类型的数据时, 必须在查询时指定这个 nested 对象, 否则查询失败)
```json
# 查询nested类型的数据,必须指定 nested.path 后再进行查询
POST my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            {"match": {"user.first": "John"}},
            {"match": {"user.last": "Smith"}}
          ]
        }
      }
    }
  }
}

# 查询错误,nested类型的数据在查询时指定这个对象,否则无法查询结果
POST my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"user.first": "John"}},
        {"match": {"user.last": "Smith"}}
      ]
    }
  }
}
```

# 搭建管理搜索功能的微服务模块 service-list
它是搜索模块服务, 负责从商品管理模块 service-product 中获取商品信息整合后存入 es
在这个模块中定义
[[ElasticSearch_核心概念#SpringData 整合 ES]]

## 商品上架接口
1. 通过远程调用 service-product 服务, 获取到 goods 实体类中的各个数据
2. 将这些数据封装到一个 goods 实体类对象中, 调用 esRepository 存入实体类对象即可完成相应操作
## 商品下架接口
通过 esRepository 调用删除方法, 通过 id 将 elastic search 中的数据删除即可

## 商品热度排名

实现步骤:
1. 在商品搜索的服务中提供更新商品热度值的接口, 并提供远程调用接口
2. 在商品详情展示服务中, 每次查询某一商品时都通过远程调用这个接口对热点值进行更新

在进行商品热度更新时, 会出现两个问题: 
1. 在搭建集群进行项目部署后, 对于共享变量 (多个线程同时访问了一个商品) 的操作需要保证安全性 (原子性)
2. Es 的更新是 IO 操作, 如果频繁进行数值的修改会大幅降低系统性能

解决方案: 
使用 Redis 做中间操作, 将热点值数据暂时存储在 Redis 中, 到达一定的阈值时再向 es 中写入一次以解决频繁 IO 的问题 (例如每次查询某个商品时, 都在 Redis 中对他的 hotScore 自增 1, 达到每 10 次访问后再一次性保存到 es 中). 
同时 Redis 的操作就是原子性的, 也能保证这个共享变量的的安全性
使用 [[Redis#Redis 中的数据类型]] 中的 Zset (sortSet) 来存储较为合适, 这个类型中自带评分, 直接修改评分即可


# 商品检索

## 所需的各个检索方式
1. 匹配查询-通过搜索框的 Keyword 关键字进行查询
2. 过滤查询-通过首页三级分类进行查询
3. 过滤查询-通过品牌进行过滤查询
4. 过滤查询-通过平台属性进行
5. 聚合查询-品牌聚合 (通过品牌进行分组)
6. 聚合查询-平台属性聚合
7. 排序查询-根据价格排序
8. 排序查询-根据热度排序
9. 分页查询
10. 高亮查询 (必须是关键字匹配查询才有高亮显示)
11. 结果集过滤 (并不是所有查询的字段都需要)

## 搜索功能分析
Web-all 对外暴露接口 --> service-list 实现查询数据并返回
