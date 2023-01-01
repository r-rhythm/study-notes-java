# 概述
> [!info] SpringData是什么
> - [[Spring]]的构成模块之一,对各种类型的数据库进行操作
> - 简化关系型、非关系型数据库、索引库的**访问**
> - 主要目标是使数据库的访问更加方便和快捷

# SpringData 定义方法的规范
> 按照 SpringData 的命名规范定义方法即可实现对数据库的简单查询,无需我们手动实现,它为我们自动封装查询条件

方法名规范
 1. 查询方法以 find/read/get 开头
 2. 加上查询条件属性,属性的首字母要大写
 3. 多个属性间添加关联条件(*And/Or/Is/Not/Between*)
 4. 例如(*findByTitleAndPrice*)
 5. **注意传入的参数要和方法的名字顺序相同**例如`List<User> findByNameAndAge(String name,Integer age);`