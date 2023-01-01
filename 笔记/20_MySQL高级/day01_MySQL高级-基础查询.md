# 补充特性
- 从 MySQL8 起, 数据库的默认字符集为 utf-8mb4, 避免中文乱码集 emoji 存储问题
- 在 Window 环境中 MySQL 不区分大小写, Linux 环境中, 数据库名/表名/表别名/变量名严格区分大小写
- 严格模式中 select 后只能出现 group by 中的分组字段及聚合函数

# 账户与权限管理
```sql
# 授权该用户 atguigu 表下的所有权限
GRANT ALL PRIVILEGES ON atguigudb.* TO 'wei'@'%';

# 回收用户对这个表的删除权限
REVOKE DELETE ON atguigudb.* FROM 'wei'@'%';

# 显示当前用户的所有权限
SHOW GRANTS;
```

# 逻辑架构
## 逻辑架构图
![[MySQL-执行流程.png]]
## MySQL 架构组件
- 连接池, 提供多个用于客户端与服务器端进行交互的连接
- SQL 接口 (SQL interface), 接收 SQL , 并向客户端返回结果
- 解析器 (Parser), 对 SQL 语句进行语法解析, 将 SQL 解析为一颗语法树
- 优化器 (**Optimizer**), SQL 核心组件, 对 SQL 语句进行优化并生成执行计划
- 插件式存储引擎 (*Pluggable Storage Engines*), 不同的引擎定义了底层文件存储时与系统的交互模式
## SQL 执行流程
1. 客户端与服务端建立 tcp 连接, 并检查用户权限等
2. 通过 SQL 接口接收发送的 SQL 语句, 此时首先检查缓存是否命中, 如果命中直接返回结果
3. 解析器对 SQL 语句进行解析并生成一棵对应的解析树, 此时会根据 SQL 的规则检查这个解析树是否合法, 例如表/字段是否存在等
4. MySQL 优化器对我们发送的 SQL 语句进行一些优化, 以期待将查询的 IO 成本和 CPU 成本降到最低并生成一个执行计划
5. 执行器调用存储引擎接口执行 SQL 查询并将结果存入到缓存中后返回数据给客户端

## 存储引擎
不同的存储引擎存储在本地的文件各不相同
- InnoDB 存储引擎
	- MySQL 默认的存储引擎
	- 支持事务
	- 支持[[行锁]]
	- 支持外键
- MyISAM 区别于 InnoDB 的方面
	- 不支持外键/事务
	- 表锁, 即使操作一条数据也会将整个表进行上锁, 不利于高并发操作
	- 只缓存索引, 不缓存真实数据
	- 系统自带的表使用

# JOIN 连接查询
## 数据准备
```sql
CREATE TABLE `t_dept` (
 `id` INT NOT NULL AUTO_INCREMENT,
 `deptName` VARCHAR(30) DEFAULT NULL,
 `address` VARCHAR(40) DEFAULT NULL,
 PRIMARY KEY ( `id`)
);
 
CREATE TABLE `t_emp` (
 `id` INT NOT NULL AUTO_INCREMENT,
 `name` VARCHAR(20) DEFAULT NULL,
 `age` INT DEFAULT NULL,
 `deptId` INT DEFAULT NULL,
`empno` INT NOT NULL,
 PRIMARY KEY (`id`),
 KEY `idx_dept_id` (`deptId`)
 #CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) REFERENCES `t_dept` (`id`)
);

INSERT INTO t_dept(id,deptName,address) VALUES(1,'华山','华山');
INSERT INTO t_dept(id,deptName,address) VALUES(2,'丐帮','洛阳');
INSERT INTO t_dept(id,deptName,address) VALUES(3,'峨眉','峨眉山');
INSERT INTO t_dept(id,deptName,address) VALUES(4,'武当','武当山');
INSERT INTO t_dept(id,deptName,address) VALUES(5,'明教','光明顶');
INSERT INTO t_dept(id,deptName,address) VALUES(6,'少林','少林寺');

INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(1,'风清扬',90,1,100001);
INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(2,'岳不群',50,1,100002);
INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(3,'令狐冲',24,1,100003);

INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(4,'洪七公',70,2,100004);
INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(5,'乔峰',35,2,100005);

INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(6,'灭绝师太',70,3,100006);
INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(7,'周芷若',20,3,100007);

INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(8,'张三丰',100,4,100008);
INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(9,'张无忌',25,5,100009);
INSERT INTO t_emp(id,NAME,age,deptId,empno) VALUES(10,'韦小宝',18,NULL,100010);
```
## 内连接查询
查询两表交集 (左/右表都存在的数据)
 `table1 INNER JOIN talbe2 ON table1.id = table2.table1_id`
```sql
# 需求1：查询所有有部门的员工信息以及他所在的部门信息
SELECT * FROM t_dept AS d INNER JOIN t_emp AS e ON d.id = e.deptId;
```
## 左连接查询 
查询左表全集 (左表的所有信息, 右表以 Null 补充)
```sql
# 需求2：查询所有用户，并显示其部门信息（如果员工没有所在部门，也会被列出）
SELECT * FROM t_emp AS e LEFT JOIN t_dept AS d ON e.deptId = d.id;
```
## 右连接查询
查询右表全集 (左表以 Null 作为补充)
```sql
# 需求3：列出所有部门，并显示其部门的员工信息（如果部门没有员工，也会被列出）
SELECT * FROM t_emp AS e RIGHT JOIN t_dept AS d ON e.deptId = d.id;
```
## 连接条件查询
左右连接查询设置查询条件查询
```sql
# 需求4：查询没有加入任何部门的员工（先查询所有员工，再过滤掉包含部门的数据）
SELECT * FROM t_emp AS e LEFT JOIN t_dept AS d ON e.deptId = d.id WHERE d.id IS NULL; # 注意,查询一个字段是否是空使用 IS NULL,不要使用 filed = NULL;

# 需求5：查询没有任何员工的部门 => 查询B且不包含A
SELECT * FROM t_emp AS e RIGHT JOIN t_dept AS d ON e.deptId = d.id WHERE e.deptId IS NULL;
```
## 联合查询
MySQL 不支持 FULL JOIN (查询左右表的全部结果), 用 LEFT JOIN + UNION + RIGHT JOIN 
联合查询实现
```sql
# 需求6：查询所有员工和所有部门 => 左右表全部结果
SELECT * FROM t_emp AS e LEFT JOIN t_dept AS d ON e.deptId = d.id
UNION
SELECT * FROM t_emp AS e RIGHT JOIN t_dept AS d ON e.deptId = d.id;

# 需求7：查询没有加入任何部门的员工，以及查询出部门下没有任何员工的部门 => A的独有+B的独有
SELECT * FROM t_emp AS e LEFT JOIN t_dept AS d ON e.deptId = d.id WHERE d.id IS NULL
UNION
SELECT * FROM t_emp AS e RIGHT JOIN t_dept AS d ON e.deptId = d.id WHERE e.deptId IS NULL;
```
> 注意:
> - UNION 联合查询要求字段与顺序一致
> - 如果确定两表结果不会重复，则使用 UNION ALL 提升效率


*在做连接查询时, 请先分析*
1. 基于业务梳理需要对那些表进行关联查询
2. 通过所需数据来确定表的连接方式
3. 找出这些要进行关联查询的关联字段并将他们建立联系
