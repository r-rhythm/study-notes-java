# 搜索的概念
搜索分为两类
- 传统方式搜索:在关系型数据库中,根据关键字进行模糊查询,数据量越大性能越差,like 匹配的结果也不适用
- 搜索引擎方式(*倒排索引*)

# [[ElasticSearch_核心概念]] 简介


# es 安装
安装三个组件
1. es 服务
2. es 服务中需安装 IK(*IKAnalyzer*)分词器
3. 客户端图形化界面工具 Kibana(*可选*)
4. 进入 [测试](http://localhost:9200/)

安装 Kibana 注意事项
- 修改 kibana.yaml 文件

IK 分词器
- ik_max_word:对文本做最细粒度拆分
- ik_smart:粗粒度智能拆分

# es 关系对应

1. 索引库 Index -> 数据库 Database
2. 类型 Type -> 表 Table

| 版本 | Type |
| --- | :---: |
| 5.x | 支持多种 type |
| 6.x | 只能有一种 type |
| 7.x | **默认不再支持自定义索引类型（默认类型为：_doc）**|

4. 文档 Document <=> Row
5. 字段 Field <=> Column

***es 的一切设计都是为了提高搜索的性能***

# es 基本操作
## 索引操作
```Json
# 创建索引
PUT /my_index01

# 查看所有索引
GET /_cat/indices?v

# 查看单个索引信息
GET /my_index01

# 删除索引
DELETE /my_index01
```

## 文档操作

```json
# 创建文档
PUT /{索引名称}/{类型}/{id}
{
    jsonbody
}

# 查看文档
GET /{索引名称}/{类型}/{id}

# 修改文档
PUT /{索引名称}/{类型}/{id}
{
    jsonbody
}

# 修改局部属性
POST /{索引名称}/_update/{docId}
{
  "doc": {
    "属性": "值"
  }
}
```

## 映射
> es 中的映射就类似于数据库中的表结构,使用它来对每个字段进行约束

动态映射:根据文档字段自动识别类型,称为自动映射

静态映射:手动添加映射

# DSL 高级查询 (重点)

注意, 这里查询都是==使用 POST 请求==

## 匹配所有
```json

# 查询所有文档
POST /my_index
{
    "query":{
        "match_all":{}
    }
}
```

## 匹配查询 (查询出所有分词后符合条件的数据)
```json

# 匹配查询
POST /my_index/_search
{
  "query": {
    "match": {
      "title": "华为"
    }
  }
}

```

## 多字段匹配
```json
# 多字段匹配
POST /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "vivo",
      "fields": ["title","category"]
    }
  }
}
```

## 前缀匹配
```json
# 前缀匹配
POST /my_index/_search
{
  "query": {
    "prefix": {
      "title": {
        "value": "vivo"
      }
    }
  }
}

```

## 精确查询 term 
表示在进行查询时不会对这个查询条件做分词处理, 会将他作为一个整体作为条件查询
如需要 term 匹配多个词, 可以使用 terms 

## 范围查询
```json
# 查询范围
POST /my_index/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 3000,
        "lte": 5000
      }
    }
  }
}
```

## 返回指定字段的结果
在"query"同级别添加 source
```json
# 指定返回字段
POST /my_index/_search
{
  "query": {
    "match_all": {}
  }
  , "_source": ["title","price"]
}
```

## 组合查询
bool 各条件之间有 and,or 或 not 的关系
- must: 各个条件都必须满足，所有条件是 and 的关系
- should: 各个条件有一个满足即可，即各条件是 or 的关系
- Must_not: 不满足所有条件，即各条件是 not 的关系; 排除
- filter: 与 must 效果等同，但是它不计算得分，效率更高点。

```json
# must
POST /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "华为"
          }
        },
        {
          "range": {
            "price": {
              "gte": 1000,
              "lte": 6000
            }
          }
        }
      ]
    }
  }
}
```

```json
# should
POST /my_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": "华为"
          }
        },
        {
          "range": {
            "price": {
              "gte": 3000,
              "lte": 5000
            }
          }
        }
      ]
    }
  }
}

# 如果should和must同时存在，他们之间是and关系

POST /my_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": "华为"
          }
        },
        {
          "range": {
            "price": {
              "gte": 3000,
              "lte": 5000
            }
          }
        }
      ],
      "must": [
        {
          "match": {
            "title": "华为"
          }
        },
        {
          "range": {
            "price": {
              "gte": 3000,
              "lte": 5000
            }
          }
        }
      ]
    }
  }
}
```

```json
# must not(表示必须不包含此条件)
POST /my_index/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "title": "华为"
          }
        },
        {
          "range": {
            "price": {
              "gte": 4900,
              "lte": 5000
            }
          }
        }
      ]
    }
  }
}
```

```json
# filter 过滤,与must作用相同, 但是它不计算评分排序所以效率会更高
POST /my_index/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "match": {
            "title": "华为"
          }
        }
      ]
    }
  }
}
```

## 聚合查询
- max
- min
- avg
- sum
- stats 查询匹配数据的所有聚合数据,包括最大值最小值等等
- terms 桶聚合相当于 sql 中的 group by 语句

## 查询结果排序
```json
POST /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "华为"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "price": {
        "order": "asc"
      }
    },
    {
      "_score": {
        "order": "desc"
      }
    }
  ]
}
```

## 分页查询
1. 分页的两个关键属性: from、size。
    - from: 当前页的起始索引，默认从 0 开始。 from = (pageNum - 1) * size
    - size: 每页显示多少条
2. scoll 分页
    滚动查询,如果要获取的数据特别多,那么可以分批查询数据

## 高亮查询
查询到的结果关键字可以设置标签以显示不同的颜色, 称为高亮查询

```json
POST /my_index/_search
{
  "query": {
    "match": {
      "title": "华为"
    }
  },
  "highlight": {
    "pre_tags": "<b style='color:red'>",
    "post_tags": "</b>",
    "fields": {
      "title": {}
    }
  }
}
```


## 近似查询(*纠错查询*)
返回包含与搜索字词相当的结果,即时搜索关键词写错了,也能查询到结果; *仅适用于英文环境*
fuzzy(模糊)
基本纠错规则
- 字母长度 0~2 个 必须完全匹配
- 3~5 允许匹配一个字符错误
- &gt;5 允许匹配两个错误

<br/>

# DSL 高级查询详解

## Bool 布尔查询 (组合查询) 


把多个条件组合在一起进行查询, 它将多个子查询组合为一个布尔表达式 (boolQuery), 多个子查询间的逻辑关系是==与 (And)==
只有当某一个文档 (\_doc) 满足布尔查询下所有子查询的条件, es 才认为该文档满足查询条件. 一个 boolQuery 下可通过不同的匹配类型嵌套多个boolQuery
布尔查询支持的子查询有四种不同的匹配类型, 分别是:
- Must: 文档必须匹配 must 子句下的条件
- Should: 通常情况下是数组字段. 默认情况下, 匹配的文档必须满足 should 子句下的至少一个条件
- Must_not: 文档必须不包含这个子句下的条件
- Filter: 过滤器, 文档必须匹配这个过滤条件. Filter 与 must 的唯一区别是, filter 不会影响查询的评分 (score)


## Aggregations 聚合查询 

简写为 aggs. 类似 [[MySQL]] 中的分组查询
Es 中的桶 (bukets) 就相当于 MySQL 中的一个分组, 每个桶代表了一组相同特征的数据
Es 聚合查询的基本语法结构:
- Aggregation_name: 代表这一组聚合查询的名称, 可以随意命名, 后续可以通过这个名称在查询结果中获取需要的计算结果
- Aggregation_type: 聚合类型, 表示我们想要如何统计数据. 聚合类型主要有两大类型: 桶聚合和指标聚合
<br/>

桶聚合 (buket): 代表了一组相同特征的数据.
- Terms 类似 MySQL 中的 group by, 根据字段的唯一值分组
- Histogram 根据数值间隔进行分组, 例如商品的价格按每 100 进行分组, 0 100 200 300...
- Date Histogram 按照时间间隔进行分组, 每月/每天/每小时
- Range 按照数据范围进行分组, 如 0-150 为一组, 151-270 为一组等
- *桶聚合通常配合指标聚合一起使用, 如果不指定特定的指标聚合信息, 默认使用 value_count 统计桶内的文档总数*
<br/>

指标聚合: 类似 MySQL 中的聚合函数, 指标聚合可以单独使用, 也可配合桶聚合使用. 常用的统计函数如下:
- Value_count 统计总数
- Cardinatily 类似 MySQL 中的 distinct 字段, 只统计不重复数据总数
- Avg 求平均值
- Sum 求综合
- Max 求最大值
- Min 求最小值

默认情况下, es 会根据 doc_count 文档总数进行降序排序
