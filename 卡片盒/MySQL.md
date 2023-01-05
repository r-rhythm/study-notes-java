# 简介
SQL (Structure Query Language) 结构化查询语言

# 暂存盒
- MySQL 数据库本质就是文件存储系统
- 结果去重 distinct 将查询
- 到的 select distinct sm.* from 
- limit(m,n) m 表示此页的数据位置索引(从第几条数据开始)计算公式是为 ==(当前页-1)* 每页显示条数==
- 当表中的 value 不唯一,可以查询出它的上级(*父字段*) id,再通过 parent_id + value 做到精确查询
- CONCAT() 将多个字符串进行拼接
- GROUP_CONCAT (expr) 函数: 能够在分组之后, 能够将同一个组中的多行数据按照指定的规则拼接在一起进行显示;(参数一: 处理的字段, 参数二: 排序的字段, 参数三: 分隔符)

## 指定返回条数
- LIMIT ? OFFSET ?,指定要返回多少条结果,和从哪一行开始返回(从0开始,目标条数-1)
```sql
SELECT 
    employee_id, first_name, last_name
FROM
    employees
ORDER BY first_name
LIMIT 5 OFFSET 3; # 从第 4 条记录开始,返回 5 条数据
```

## 去重
- UNION 联合查询
- DISTINCT 结果去重关键字

# 多表联查
