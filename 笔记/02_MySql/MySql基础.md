# 为什么要用 MySQL
- 使用 MySQL 保证[[事务 Transaction]]的一致性,也就是保证数据的一致性
## 基本

### SQL语句分类

```mysql
1. 数据定义语言：简称DDL(Data Definition Language)，用来定义数据库对象：数据库，表，列等。关键字：create，alter，drop等 

2. 数据操作语言：简称DML(Data Manipulation Language)，用来对数据库中表的记录进行操作。关键字：insert，delete，update等

3. 数据控制语言：简称DCL(Data Control Language)，用来定义数据库的访问权限和安全级别，及创建用户。

4. 数据查询语言：简称DQL(Data Query Language)，用来查询数据库中表的记录。关键字：select，from，where等
    
5. 其他语句：USE语句，SHOW语句，SET语句等。这类的官方文档中一般称为命令。
```

### SQL中的数据类型

| **类型名称**          | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| int（整数长度）       | 整数类型，默认长度11                                         |
| double（m,d）         | 小数类型                                                     |
| decimal（m,d）        | 指定整数位与小数位长度的小数类型                             |
| date                  | 日期类型，格式为yyyy-MM-dd，包含年月日，不包含时分秒  2020-01-01 |
| datetime              | 日期类型，格式为 YYYY-MM-DD HH:MM:SS，包含年月日时分秒   到9999年 |
| timestamp             | 日期类型，时间戳  从1970年到2038年                           |
| varchar（字符串长度） | 文本类型， M为0~65535之间的整数                              |



## MySQL基本语法

### DDL

#### 创建数据库

```mysql
1.关键字:create database
2.语法:
  create database 数据库名 charset utf8;
```

#### 查看数据库

```mysql
#查看所有数据库
show databases;

#查看指定数据库的定义信息
show create database 数据库名
```

#### 删除数据库

```mysql
1.关键字:drop
2.语法:
  drop database 数据库名字
```

#### 切换数据库

```mysql
use 数据库名;
```

####  创建表

```mysql
1.关键字:create table
2.语法:
  create table 表名(
    列名 数据类型(长度) [约束],
    列名 数据类型(长度) [约束],
    列名 数据类型(长度) [约束]
  );
 
4.注意:
  a.如果定义完一个列之后,后面还有其他的列,需要加逗号
  b.如果定义最后一个列了,就不用加
  c.一张表应该有一个主键(约束),  primary key
    当一列设置成主键了,此列中的数据唯一,不能重复
    一个主键可以代表一条数据(好比是身份证)
```

#### 查看表

```mysql
#查看所有表
show tables;

#查看表结构
desc 表名; -- description
```

#### 删除表

```mysql
1.关键字:drop table
2.语法:
  drop table 表名
```

#### 修改表结构

```mysql
alter table 表名 add 列名 类型(长度) [约束];
作用：添加列. 
```

### DML[重点]

#### 插入数据

```mysql
1.关键字:insert into values
2.语法:
  insert into 表名(列名1,列名2,...) values (列值1,列值2)
```

#### 删除数据

```mysql
1.关键字:delete from
2.语法:
  delete from 表名-> 一次全删除
  delete from 表名 where 条件-> 按照条件删除
```

#### 修改数据

```mysql
1.关键字:update set
2.语法:
  update 表名 set 列名 = 值 -> 是将指定的列所有的数据全部修改
  update 表名 set 列名 = 值 where 条件 -> 根据条件修改数据
```



## 简单查询[重点]

### 简单查询

```mysql
1.关键字:select from where
2.语法:
  select 列名,列名 from 表名 [where条件]
  
  select * from 表名-> 查询表中所有列的数据  * 代表的是查询结果展示所有的列
  select 列名 from 表名-> 查询指定这一列的数据   最终查询结果只展示指定的一列
  select 列名 from 表名 where 条件-> 按照指定的条件查询数据
  
3.注意:
  a.select后面写多少列,查询结果就展示多少列的数据
    写哪些列,查询结果就展示哪些列的数据
  b.查询出来的结果是一张伪表,这个表的数据是只读的,不能动
```

### 条件查询

* BETWEEN  ...AND... *显示在某一区间的值(含头含尾)*
* 字段 IN(set) *查询该字段中值在set内的结果* `查询id为1,3,7的商品: id  in(1,3,7)`
* LIKE *模糊查询*
  * % *代表任意个任意字符*
  * _ *代表任意一个字符*
* IS NULL *判断是否为空* `IS NOT NULL判断非空`
* AND *多个条件都满足才为true*
* OR *有真则真*
* NOT *不成立则为真*

### 排序查询

```mysql
1.关键字:order by  desc|asc
2.语法:
  select 列名 from 表名 [where 条件] order by 排序字段 asc|desc-> 查询之后要指明对哪一列进行排序
  
  asc:升序(默认)
  desc:降序
      
3.先查询,后排序
```

#### 书写sql语句关键字的顺序

```mysql
select 		列名		5
from 		表名		1
where 		条件		2
group by 	分组		3
having 		分组条件   4	
order by	排序		6

执行顺序:
from 
where 
group by 
having 
select 
order by

先定位到要查询哪个表,然后根据什么条件去查,表确定好了,条件也确定好了,开始利用select查询
查询得出一个结果,在针对这个结果进行一个排序
```



## 单表查询

### 聚合查询

```mysql
1.作用:纵向操作数据
2.聚合函数:
  count(列名):统计表中有多少条数据
  sum(列名):针对指定列进行求和
  avg(列名):针对指定列求平均值
  max(列名):求指定列的最大值
  min(列名):求指定列的最小值
3.格式:
  select 聚合函数(列名) from 表名 where 条件
```

### 分组查询

```mysql
1.关键字:group by 列名  -> 根据哪一列进行分组
2.语法:
  select 列名,列名... from 表名 group by 分组字段 having 分组条件
3.分组按照哪个字段去分,小窍门:
  相同字段名为一组,不同字段名的单独为一组展示,我们就可以用该字段【相同字段名】进行分组
4.关键字:在分组之后进行条件筛选
  having 条件
5.having和where的区别:
  having:是在分组之后查询
  where:在分组之前查询
```

### 分页查询

```mysql
1.语法
  select * from 表名 limit m,n
    m:代表每页的起始位置 -> 从0开始
    n:页面显示的条数
    
2.每一页的起始位置算法:
   (当前页-1)*每页显示条数

-- 后台计算出页码、页数(页大小)
-- 分页需要的相关数据结果分析如下,
-- 注意:下面是伪代码不用于执行
int pageNum = 2; -- 当前页数
int pageSize = 5; -- 每页显示数量
int startRow = (pageNum - 1) * pageSize; -- 当前页, 记录开始的位置(行数)计算
int totalSize = select count(*) from products; -- 记录总数量
int totalPage = Math.ceil(totalSize * 1.0 / pageSize); -- 总页数
                总页数 = (总记录数/每页显示条数)向上取整
```



## 多表查询

### 交叉查询

```mysql
1.格式:
  select 列名 from 表A,表B;
2.注意:
  交叉查询会出现"笛卡尔乘积"(所有的情况都组合一遍)
 -- 查询商品的详细信息,出现了笛卡尔乘积,查询结果错误
SELECT * FROM category,products;


SELECT * FROM category,products WHERE category.cid = products.category_id;

-- 给表起别名

SELECT * FROM category c,products p WHERE c.cid = p.category_id;
```

### 外连接

```mysql
1.关键字:outer join -> outer 可省略
2.语法:
  a.左外连接:select 列名 from 表A left outer join 表B on 条件
  b.右外连接:select 列名 from 表A right outer join 表B on 条件
3.如何区分谁是左表,谁是右表
  看join这个单词
  
  在join左边的就是左表
  在join右边的就是右表
  
4.左外连接,右外连接,内连接

  a.左外连接:查询的是和右表的交集,以及左表的全部(和右表的交集,以及除交集之外的其他左表数据)
  b.右外连接:查询的是和左表的交集,以及右表的全部(和左表的交集,以及除交集之外的其他右表数据)
  c.内连接:查询的是两个表的交集
  
  /*
   左表:category
   右表:products
*/
SELECT * FROM category c LEFT JOIN products p ON c.`cid` = p.`category_id`;

SELECT * FROM category c RIGHT JOIN products p ON c.`cid` = p.`category_id`;

-- 隐式内连接查询

SELECT * FROM category c,products p WHERE c.`cid` = p.`category_id`;
```

### 子查询

```mysql
1.概述:一条查询语句作为另外一条查询语句的条件使用
2.语法:
  select 列名 from 表名 where (select 列名 from 表名)
```

