# ES 概述
Elasticsearch (简称 ES) 是一个分布式、RESTful 风格的搜索和数据分析引擎。
它是一个全文检索引擎, 默认情况下对所有的数据进行所有存储

ES 特点:
1. 查询和分析
2. 查询速度极快 (*近实时搜索, 数据 1s 之内可见*)
3. 使用倒排索引
4. ElasticSearch 基于 Lucene 搭建, Lucene 是一个全文搜索的核心工具包
5. Es 只支持 [[Json]] 格式

# 倒排索引
- 倒排索引会将新添加的文档内容进行分词, 每个词汇都会与他所出现的 DocID 对应
- 先根据关键词查出所有出现该关键词的 DocID 列表, 再从文档中查询并返回结果
> [!info] 快速理解倒排索引
> 通俗的来讲, 普通索引是通过文档来找关键字, 而倒排索引是通过关键字来找文档, 再从文档中获取具体的需要的信息

倒排索引检索操作分为三步
1. **分词**: 将一条搜索语句按照分词规则拆分为多个关键字
2. 分词后**匹配**: 使用拆分后的多个关键字进行数据库查询
3. 匹配之后的**排序**: 根据匹配次数排序

# 核心概念
- 索引 (Index) 一个索引就是一个拥有几分相似特征的文档的集合, 类似目录, 通过索引可以提高搜索的效率
- 类型 (Type) 一个索引内可以定义多个类型, 每个类型是这个索引内的逻辑上的分区
- 文档 (Document) 一个文档就是一个可以搜索的基础信息单元, 即一条信息
- 字段 (Field) 一个文档可以通过不同的属性进行分类识别, 它对应了数据库中的字段
- 映射 (Mapping) 对处理数据的方式或规则做一些约束, 类似于数据库的表结构, 它约束了某个字段的数据类型, 默认值等

# ES 的特性

## 文档搜索
- 倒排索引被创建后是不可改变的, 它无法被修改
- 若需要一个新的文档被索引到则需要重构整个索引, 所以想要在保留不变性的前提下完成倒排索引的关跟新就需要使用动态更新索引

## 动态更新索引
如何在保留不变性的前提下实现倒排索引的更新?
- 通过=="补充索引"==的形式加入新的索引来反应修改, 每一个倒排索引都会被轮流查询到，从最早的开始查询完后再对结果进行合并。类似于字典中的附录
- Java 通过数据段 (*Segment*) 的方式来增加补充索引, 每一段本身都是一个倒排索引

## 近实时搜索
ES 通过使用缓存 + 缓冲进行读写从而达到近乎实时搜索
1. ES 写操作时, 并不会马上将数据写入硬盘, 而是将他写入缓存中, 并把操作记录到日志表 (*Translog*) 内
3. 没有写到缓存前, 他不可查, 写完后就对外可见了
4. 缓存存储的数据, 默认 30 分钟刷新到硬盘内
5. 如果文件系统缓存数据在没有刷新到硬盘时宕机了, 可以从 translog 中恢复数据到磁盘即可保证数据的安全性

## 段合并
由于自动刷新流程每秒会创建一个新的段，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。每一个段都会消耗文件句柄、内存和 cpu 运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。
- Elasticsearch 通过在后台进行段合并来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段。
- 段合并的时候会将那些旧的已删除文档从文件系统中清除。被删除的文档（或被更新文档的旧版本）不会被拷贝到新的大段中。
- 启动段合并不需要你做任何事。进行索引和搜索时会自动进行。


# 分词器 Analyzer
分词器是用来将一段文本分析成为一个一个的词, ElasticSearch 引擎再根据这些词去做倒排索引

## 内建分词器 
-  `Standard Analyzer`  标准分词器 (默认), 可以对英文单词进行拆分处理,  分词后所有字母均为为小写
- `simple` 简单按照空格进行分词, 保留大写字母
- `stop` 去掉无意义词, 例如: the/a/an 等, 分词后所有字母均为为小写
- `keyword` 将输入的文本作为一个关键字, 不对其进行拆分

## Ik 分词器
Es 中内建的所有分词器都无法对中文分词做到有效的处理, 所以需要安装一个支持中文分词的分词器, 这就是 ik 分词器

安装步骤:
	*linux+docker*
1. [下载 ik 分词器](https://github.com/medcl/elasticsearch-analysis-ik)
2. 上传压缩包至 linux, 解压 `unzip elasticsearch-analysis-ik-7.8.0.zip -d ik-analyzer`
3. 上传至 es 容器中 `docker cp ./ik-analyzer es容器id:/usr/share/elasticsearch/plugins`
4. 重启 es `docker restart elasticsearch_id`


# DSL 高级查询
Domain Specific Language (领域专用语言), Elasticsearch 提供了基于 JSON 的 DSL 来定义查询。
语法详情参见 [[ElasticSearh_语法#DSL 高级查询(重点)]]

[[ElasticSearch_核心概念]]

# 常见问题
## 为什么要使用 ES
- 虽然在关系性数据库内可以建立索引提高效率, 但在海量的数据面前, 即使建立了索引查询效率也不会太高
- 使用索引库将经常查询的字段存储在内存中, 提高数据查询的效率
- ES 不仅仅是搜索，还有分析。ElasticSearch 的相关性匹配能够比数据库的模糊查询更加智能化的返回用户想要得到的结果。例如用户想搜索 `莎士比亚` 但是输入了 `莎士笔亚`，在采用关系型数据库的情况下，即使使用模糊查询 `like %莎士笔亚% ` 也无法胜任，而 ES 的相关性匹配则能够帮助我们优化为 `莎士比亚` 并返回用户需要的结果。

## Master 的选举流程
> 7. X 版本后的 es, 是 Raft 的实现

Raft 选举流程
1. 通过选举超时时间 (*先返回信息者*) 来决定哪个节点作为候选人
2. 超时时间最短的节点作为候选人
3. 其他 follower 进行投票, 得票超过半数, 候选人作为 master
4. 补充: 详情流程[参见](https://cloud.tencent.com/developer/article/1705472)

特殊情况
- 在 master 节点应为网络等原因, 其他节点无法连接上就认为 master 挂了
- 由其他节点发起选举, 推选新的 master
- 若之前的 master 再次连接上, 不能再当 master, 需要从 follower 做起

## ES 集群的脑裂问题

指同一个集群中的不同节点对集群的状态有了不同理解, 存在两个 master

脑裂问题产生的原因:
- 网络问题: 集群间的网络延迟导致一些节点无法联系上 master, 从而选举出新的 master
- 节点负载: 当主节点又作为数据节点操作数据时, 访问量较大时会导致主节点响应迟慢, 副节点认为主节点挂掉, 选举新的 master
- 内存回收: es 数据节点进程占用的内存较大, 引起 JVM 大规模回收内存, 造成 es 失去响应

## 集群脑裂的结局方案
1. 减少误判: 增大响应时间, 默认是三秒 master 无响应认为主节点 down 机, 调大参数可以减少误判
2. 选举触发: 约定选举的触发机制, 多少个节点认为主节点挂掉才重新发起选举
3. 角色分离: 主节点只做管理, 不做数据处理

# 提高 ES 搜索速度的优化
## 硬件优化
ES 重度依赖磁盘, 磁盘越快 ES 的性能就越高, 推荐优先使用固态

## 分片策略
- 合理分片分片数 **节点数 <= 主分片数 * (副本数 + 1)**
- 推迟分片分配时间, 如果主节点无法联系上, 推迟重新选举主节点的时间

## 内存分配
- 主节点内存不用太大, 合理的分配主副节点的内存

## 写入速度优化
ES 提供了 Bulk API 支持批量操作，当我们有大量的写任务时，可以使用 Bulk 来进行批量写入。

通用的策略如下：Bulk 默认设置批量提交的数据量不能超过 100M。数据条数一般是根据文档的大小和服务器性能而定的，但是单次批处理的数据大小应从 5MB～15MB 逐渐增加，当性能没有提升时，把这个数据量作为最大值。

# SpringData 整合 ES

## [[SpringData]] 集成 es 的两种方式
1. ~~ElasticsearchRestTemplate~~ 
2. 创建 dao 接口继承 ElasticsearchRepository<映射类, 主键类型>，直接调用封装的方法实现
3. 配置类, 继承 `AbstractElasticsearchConfiguration` 重写父类的方法, 创建高级客户端
4. [参见spring.io](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#elasticsearch.clients)

## 框架集成
1. 引入依赖 `spring-boot-starter-data-elasticsearch`
2. 配置 es 的服务地址和端口号
3. 创建 document 映射类

映射实体类通过注解来声明字段的映射属性，在涉及实体类时, 需要注意以下多个注解：
 1. @Document 作用在类，标记实体类为文档对象，一般要指定以下三个属性 
	 1. IndexName: 对应索引库名称  
	 2. Shards: 分片数量
	 3. Replicas: 副本数
 2. @Id 作用在成员变量，标记一个字段作为 id 主键
 3. @Field 作用在成员变量，标记为文档的字段，并指定字段映射属性. 它的常用映射属性由以下几个:
	 1. `FieldType.Keyword` 标识一个不可分词的关键字字段
	 2. `FieldType.Text` 标识一个普通文本字段, 进行分词处理
	 3. `FieldType.数据类型` 标识这个属性对于的字段数据类型
例如电商项目中的映射实体类:
```java
@Data  
@Document(indexName = "goods" , shards = 3,replicas = 2)  
public class Goods {  
    /**  
     * 商品Id skuId  
     */    
    @Id  
    private Long id;  
  
    @Field(type = FieldType.Keyword, index = false)  
    private String defaultImg;  
  
    //  es 中能分词的字段，这个字段数据类型必须是 text！如果标识为 keyword 则不做分词处理！  
    @Field(type = FieldType.Text, analyzer = "ik_max_word")  
    private String title;  
  
    @Field(type = FieldType.Double)  
    private Double price;  
  
    //  @Field(type = FieldType.Date)   6.8.1  
    @Field(type = FieldType.Date,format = DateFormat.custom,pattern = "yyyy-MM-dd HH:mm:ss")  
    /**  
     * 新品  
     */  
    private Date createTime;  
  
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


可以通过 [[SpringData#SpringData 定义方法的规范]]来快速实现一些基础的增删改查

# ES 集群
## 集群 Cluster
一个 Elasticsearch 集群有一个唯一的名字标识，这个名字默认就是”elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

## 节点 Node
集群中包含很多服务器，一个节点就是其中的一个服务器。作为集群的一部分，它存储数据，参与集群的索引和搜索功能。
一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。

## 集群的节点类型
主节点 (*master*) 负责管理索引/维护元数据/管理集群状态, 副节点 (*data node*) 负责数据管理

## 分片 Shards ※
将数据依照一定的规则, 分别存储在不同的节点中以保障数据的安全性

### 分片控制流程※
#### 读流程
![[Es-分片读流程.png]]
1. 接收请求后, 随机选择一个数据节点为协调节点
2. 协调节点广播请求, 通知其他节点是否有客户端要查询的数据
3. 每个分片返回对应的信息给协调节点
4. 协调节点将所有数据汇总后, 返回给客户端

#### 写流程
![[Es-分片写流程.png]]
**新建和删除操作都是写操作, 必须在主分片上完成操作和才会被复制到相关的副分片**
1. 客服端发送请求, 这个请求会随机分配给一个节点, 接收请求的节点叫协调节点
2. 协调节点进行路由计算, 计算出数据所在的分片位置并将请求转发给对应的主分片机器
3. 主分片的数据修改后, 将数据同步到副本分片中
4. 所有分片处理完毕后, 协调节点将结果返回给客户端

## 副本 Replicas ※
 创建数据备份: 分片会带来一个问题, 如果某台节点 down 机, 会导致部分数据不完整 
 建立副本的目的在于, 即使某个节点挂掉, 也可以在其他节点的副本中找到这个节点的备份
**ElasticSearch 7.8. X 版本中每份索引有一个主分片和一份副本**