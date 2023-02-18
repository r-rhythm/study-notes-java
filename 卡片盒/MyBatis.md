# 简介
 > MyBatis 是一个半自动的 ORM(Object Relation Mapping)持久层框架
- ORM(Object Relation Mapping)对象关系映射,将 Java 对象与数据库中表建立映射,从而通过操作对象来影响表中数据
- POJO(Plain Old Java Object)普通 Java 对象,没有继承任何类继承,接口实现的纯净的 Java 对象
- 半自动 MyBatis & 全自动 Hibernate
  - Hibernate 全自动化持久化层框架,不用写 sql
  - MyBatis 半自动,需要自己写 sql,方便优化 sql 提高性能
- [文档参见](https://mybatis.org/mybatis-3/)

# 常见问题
1. 绑定异常可能出现的原因
	1. 三个一致没有遵守
	2. mapper 的 XML 加载问题,使用 [[MyBatis-Plus]] 代码生成器生成的 XML 文件是存放在 src-main-java 路径下的,但 maven 项目默认只会编译这个路径下的 Java 文件,这就会导致这个 XML 文件无法编译与加载
	3. 解决方案:将 xml 放到 resouces 目录下或通过配置方式去加载 xml 文件
		1. 在 pom 文件中添加配置
		2. 在 properties | yml 配置文件中添加配置
3. 在创建 `Resources.mapper.xx` XML 文件所在位置的时候,==必须一层一层的创建这个 directory 下的路劲==, 不可以用级联的方式直接. 创建, 否则他会将点也识别为文件夹名称, 踩坑...

# MyBatis 框架搭建
 [参见](https://mybatis.org/mybatis-3/zh/getting-started.html)
  - 导入相关 jar 包
  - resources 路径下编写配置文件 `mybatis-config.xml`
  - 在 `resources/mapper` 下创建名称为 `xxxMapper.xml` 映射文件(为接口中的方法映射 sql 语句)
 **要求三个一致**
      - 映射文件名称与接口名称一致
      - 映射文件中的 namespace 与接口的**全类名**一致
      - 映射文件中 sql 的 id 与接口中方法名称一致
使用核心类库 sqlSession 进行测试

# MyBatis 中配置文件
- configuration:根标签,无实际意义
- properties:引入外部 properties 属性文件,使用表达式 `${xxx.xxx}` 可在外部配置文件对配置的内容动态的进行替换
* settings:它会改变 mybatis 的运行时行为例如 `mapUnderscoreToCamelCase`开启驼峰命名
- typeAliases:为 Java 类型定义一个缩写别名,减少冗杂的全类名书写,可以为指定的类或包下所有 POJO 定义缩写名(自动使用 Bean 首字母小写的方式设置别名)
  - **在 MyBatis 中的标签结构是固定的,顺序参考[官网](https://mybatis.org/mybatis-3/zh/configuration.html#settings)**

# Mapper 映射文件
子标签共 9 个,掌握其中 8 个
  - select 定义查询
  - delete 定义删除
  - insert 定义增加
  - update 定义修改
  - sql 定义可重用的标签
  - cache 定义当前名称空间下的缓存
  - cache-ref 应用其他名称空间下的缓存配置
  - resultMap 自定义映射(类中的属性与表中的字段的映射关系)

获取主键自增数值
- 在 insert 标签内添加以下两个语句
  - `userGeneratedKeys = true`
  - `keyProperty = id`

**注意事项**
- mybaties 中事务默认不提交,如需增删改操作时,需要手动提交事务
  - sqlSession.commit()
  - sqlSession.rollback()

# Mybatis 中的参数传递※
## 单个普通参数
- Mybatis 中传递单个普通参数,可以直接入参,不用考虑参数类型及名称
## 多个普通参数
- 任意多个参数,都会**被 Mybatis 包装为一个 Map**,这个 Map 的 key 是确定的 arg0,arg1... 或 param1,param2...
- **使用一个以上的参数时,必须使用 @Param 注解标注名称,否则它会使用默认的参数名替代**
## POJO 入参
- ~~Mybatis 中传递 POJO 参数是使用 POJO 中属性获取参数~~

## 命名参数(重点)
- 需要 2~3 个参数,又不支持 POJO 入参时,使用命名参数
- 使用注解 @Param("") 为要传递的这个参数命名,Mybatis 就会使用这个命名作为 Map 中该参数的 Key
- 命名参数同时支持 `param1,param2...` 入参
   ```java
    /**
         * [测试命名参数]
         * 通过id和姓名获取员工信息
         *
         * @param id
         * @param lastName
         * @return
         */
        Employee selectEmployeeByEmpIdAndLastNameNamed(@Param("empId")int id,
                                                       @Param("lastName") String lastName);
    ```
   ```xml
    <!--    命名参数传递 通过id与name查询员工信息-->
        <select id="selectEmployeeByEmpIdAndLastNameNamed" resultType="employee">
            select
                id,last_name,email,salary
            from
                tbl_employee
            where
                id=#{empId}
            and
                last_name=#{lastName}
        </select>
    ```
## Map 参数
- 需要多个参数,又不支持 POJO 入参时使用 Map 参数
- 使用 Map 入参时,map 的 key 就是参数的 ey
   ```java
    /**
         * [测试Map参数]
         * 通过empId和lastName获取员工信息
         *
         * @param map
         * @return
         */
        Employee selectEmployeeByEmpIdAndLastNameMap(Map<String, Object> map);
    ```

 ```java
    //测试map参数查询
    Map<String,Object> map = new HashMap<>();
            map.put("empId",1);
            map.put("lastName","liuwei");
        
            Employee employee = mapper.selectEmployeeByEmpIdAndLastNameMap(map);
            System.out.println("employee = " + employee);
```

# Mybatis 中#{ }与${ }的区别
- $ 底层使用了 Statement 对象,使用 SQL 语句拼接字符串的方式传递参数,存在 SQL 注入的安全隐患
- \# 底层使用的时 preparedStatement 预编译对象,使用占位符 ? 的方式传递参数,不存在 Sql 注入安全隐患,相对安全
## $的使用场景
- 如需使用动态表名查询数据时,# 解决不了,因为 # 可以解决的是占位符位置的参数,而表名是不能使用占位符的,此时便需要使用 $

 ```java
   /**
       * [测试$使用场景]通过动态表名及id获取员工信息
       * @param tableName
       * @param empId
       * @return
       */
      Employee selectEmployeeByDynamicTable(@Param("tableName")String tableName,
                                            @Param("empId")int empId);
  ```

 ```xml
  <!--    $使用场景,根据动态表名和id查询员工-->
      <select id="selectEmployeeByDynamicTable" resultType="employee">
          select
              id,last_name,email,salary
          from
              ${tableName}
          where
              id=#{empId}
      </select>
  ```

# 查询的四种情况

- 查询单行数据返回单个对象
- 查询多行数据返回单个对象
- 查询单行数据返回 Map 结果
 ```java
  /**
       * [测试单行数据返回Map]通过id获取员工信息
       * @param empId
       * @return
       */
      Map<String,Object> selectEmpByEmpIdResultMap(int empId);
  ```

 ```xml
  <!--    查询单行数据返回map-->
      <select id="selectEmpByEmpIdResultMap" resultType="map">
          select
              id,last_name,email,salary
          from
              tbl_employee
          where
              id=#{empId}
      </select>
  ```

- 查询多行数据返回 Map 结果:使用@MapKey 注解指定所返回的多个结果各个结果的 key 值(只要是唯一标识即可设置为@MapKey 的 value)

 ```Java
  /**
       * 查询多行数据返回map集合
       *
       * @return
       * @MapKet注解,设置map中key是哪个value值[唯一标识]
       */
      @MapKey("id")
      Map<String, Object> selectAllEmpResultMap();
  ```

 ```xml
  <!--    查询多个数据返回map-->
      <select id="selectAllEmpResultMap" resultType="employee">
          select
              id,last_name,email,salary
          from
              tbl_employee
      </select>
  ```

# Mybatis 自动映射与自定义映射(重点)

## 自动映射 resultType
- 自动将表中的字段与类中的属性建立映射关系,自动将结果集中的数据封装到对象中
- 自动映射的局限性
  - 自动映射要求字段名与属性名一致才可以映射成功\[支持开启驼峰式自动映射]
  - 自动映射只能解决单表操作
  - 如需多表连接查询 resultType 无法解决,此时可以使用 resultMap 自定义映射
  - resultType 期望返回结果的全类名或别名
  - 如果返回值是集合,resultType 应该设置为类本身而不是集合类型

## 自定义映射(重点)
### 级联映射

 ```java
  /**
       * [级联映射]
       * 通过id查询员工及所属部门信息
       *
       * @param empId
       * @return
       */
      Employee selectEmpAndDeptByEmpId(int empId);
  ```
 ```xml
  <!--    级联映射-->
      <resultMap id="empAndDept" type="employee">
  
          <id property="id" column="id"/>
          <result property="lastName" column="last_name"/>
          <result property="email" column="email"/>
          <result property="salary" column="salary"/>
  
  <!--        自定义dept映射关系-->
          <result property="dept.deptId" column="dept_id"/>
          <result property="dept.deptName" column="dept_name"/>
          <result property="dept.deptLocation" column="dept_location"/>
  
      </resultMap>
  
      <select id="selectEmpAndDeptByEmpId" resultMap="empAndDept">
          select
              e.id,e.last_name,e.email,e.salary,
              d.dept_id,d.dept_name,d.dept_location
          from
              tbl_employee e,
              tbl_dept d
          where
              e.dept_id=d.dept_id
          and
              e.id=#{empId}
      </select>
  ```

### 自定义映射-association\[多对一]
- 基础查询
```xml
<resultMap id="empAndDeptAssociationRm" type="employee">
        <id property="id" column="id"/>
        <result property="lastName" column="last_name"/>
        <result property="email" column="email"/>
        <result property="salary" column="salary"/>

    <!--        使用association自定义dept映射-->
        <association property="dept" javaType="com.atguigu.ssm.pojo.Dept">
            <id property="deptId" column="dept_id"/>
            <id property="deptName" column="dept_name"/>
            <id property="deptLocation" column="dept_location"/>
        </association>
    </resultMap>

<!--    通过员工id查询员工及所属部门信息-->
    <select id="selectEmpAndDeptByEmpId" resultMap="empAndDeptAssociationRm">

        SELECT
                e.`id`,e.`last_name`,e.`email`,e.`salary`,
                d.`dept_id`,d.`dept_name`,d.`dept_location`
        FROM
                tbl_employee e,
                tbl_dept d
        WHERE
                e.`dept_id` = d.`dept_id`
        AND
                e.`id` = #{empId}
    </select>
```

- association-分步查询
	 需求:通过员工 id 获取员工信息,及员工所属部门信息
1. 通过员工 id 获取员工信息\[部门 id: dept_id]
2. 通过\[部门 id]获取部门信息

   ```java
   /**
        * 通过部门id查询部门信息
        *
        * @param deptId
        * @return
        */
       Dept selectDeptByDeptId(int deptId);
   ```

   ```xml
   <select id="selectDeptByDeptId" resultType="dept">
           select
               dept_id,
               dept_name,
               dept_location
           from
               tbl_dept
           where
               dept_id=#{deptId}
       </select>
   ```

   ```java
   /**
        * 通过员工id获取员工及所属部门的信息[association分步查询]
        *
        * @param emptId
        * @return
        */
       Employee selectEmpAndDeptByAssociationStep(int emptId);
   ```

   ```xml
   <!--    association分步查询-->
       <resultMap id="empAndDeptAssociationStep" type="employee">
           <id property="id" column="id"/>
           <result property="lastName" column="last_name"/>
           <result property="email" column="email"/>
           <result property="salary" column="salary"/>
   
   <!--        查询出dept_id后调用DeptMapper中的查询方法继续查询部门信息-->
           <association property="dept"
                        select="com.atguigu.ssm.mapper.DeptMapper.selectDeptByDeptId"
                        column="dept_id"/>
       </resultMap>
       <select id="selectEmpAndDeptByAssociationStep" resultMap="empAndDeptAssociationStep">
           select
               id,last_name,email,salary,dept_id
           from
               tbl_employee
           where
               id=#{empId}
       </select>
   ```

### 自定义映射-collection\[一对多]
- 基础查询

```java
 /**
     * [Collection]通过部门id查询部门及该部门下所有员工信息
     *
     * @param deptId
     * @return
     */
    Dept selectDeptByDeptIdCollection(int deptId);
```

```xml
<resultMap id="deptCollection" type="dept">
        <id property="deptId" column="dept_id"/>
        <result property="deptName" column="dept_name"/>
        <result property="deptLocation" column="dept_location"/>

<!--property="empList" ofType="com.atguigu.ssm.pojo.Employee"意为empList是一个存储Employee的集合,可以同javaType指定这个类型,但在一般情况下，MyBatis 可以推断 javaType 属性,所以不必填写-->
        <collection property="empList" ofType="com.atguigu.ssm.pojo.Employee">
            <id property="id" column="id"/>
            <result property="lastName" column="last_name"/>
            <result property="email" column="email"/>
            <result property="salary" column="salary"/>
        </collection>
    </resultMap>

    <select id="selectDeptByDeptIdCollection" resultMap="deptCollection">
        select
            d.dept_id,d.dept_name,d.dept_location,
            e.id,e.last_name,e.email,e.salary
        from
            tbl_dept d,
            tbl_employee e
        where
            e.dept_id=d.dept_id
        and
            d.dept_id=#{deptId}
    </select>
```

- collection 分步查询\[将一步的多表查询,改为两步的单表查询]
 ```java
  /**
       - 通过部门 id 查询该部门下所有员工信息
       *
       - @param deptId
       - @return
       */
      List<Employee> selectEmpByDeptId(int deptId);
  ```

 ```xml
  <!--    通过部门id查询该部门下所有员工信息-->
      <select id="selectEmpByDeptId" resultType="employee">
          select
              id,last_name,email,salary,dept_id
          from
              tbl_employee
          where
              dept_id=#{deptId}
      </select>
  ```

 ```java
  /**
       - Collection 分步查询
       *
       - @param deptId
       - @return
       */
      Dept selectDeptByDeptIdCollectionStep(int deptId);
  ```

 ```xml
  <resultMap id="deptCollectionStep" type="dept">
          <id property="deptId" column="dept_id"/>
          <result property="deptName" column="dept_name"/>
          <result property="deptLocation" column="dept_location"/>
  
          <collection property="empList"
                      select="com.atguigu.ssm.mapper.EmployeeRmMapper.selectEmpByDeptId"
                      column="dept_id"/>
      </resultMap>
  
      <select id="selectDeptByDeptIdCollectionStep" resultMap="deptCollectionStep">
          select
              dept_id,
              dept_name,
              dept_location
          from
              tbl_dept
          where
              dept_id=#{deptId}
      </select>
  ```


# 延迟加载(懒加载)
 需要使用数据时再加载,未使用数据是暂时不加载
 优势: 未使用数据时,效率较高
 不足: 如数据量较大时,查询效率较低(与延迟加载无关),此时便会使用到 Mybatis 中缓存机制

## 全局设置
 在配置文件中统一设置

```xml
<setting name="lazyLoadingEnabled" value="true"/>
```

## 局部设置
- 在 association 和 collection 中设置 fetchType 属性 `lazy` 或 `eager`
- 配有局部设置的使用局部配置
   ```xml
    <collection property="empList"
                        select="com.atguigu.ssm.mapper.EmployeeRmMapper.selectEmpByDeptId"
                        column="dept_id"
                        fetchType="eager/lazy"/> 
    ```

# MyBatis 中动态 SQL[重点]

> 动态 SQL 指 SQL 可以动态变化,MyBatis 支持动态 SQL 语法,无需手动去拼接 SQL
> MyBatis 中使用各种标签实现动态 SQL,同时采用了 OGNL 表达式简化这个操作

动态 Sql 标签

- if 简单的分支判断
- where 标签,用于解决 WHERE 关键字及多出的条件问题(前面的第一个 AND 和 OR)
 > *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

- trim 如果 where 元素与你期望的不太一样，你也可以通过自定义 trim 元素来定制 *where* 元素的功能。
      - prefix 添加前缀
      - prefixOverrides 去掉前缀
      - suffix 添加后缀
      - suffixOverrides 去掉后缀

  - set 会动态地在行首插入 SET 关键字，并会删掉额外的逗号

  - choose 有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。

- foreach 循环结构

  - collection 设置需要迭代的集合或数组

  - item 设置迭代的临时变量

  - separator 设置分隔符

  - open 设置循环开始字符

  - close 设置循环结束字符

==使用 foreach 实现批量操作==
 ```java
/**  
 * 批量删除供应商信息  
 *  
 * @param 存在多个id的集合
 */  
void batchDeleteSupplier(List ids);
 ```

```SQL
<!-- 批量删除供应商 -->  
<delete id="batchRemoveSupplier">  
    DELETE    
    FROM jxc.t_supplier    
    WHERE t_supplier.supplier_id IN    
    <foreach collection="ids" item="id" separator="," open="(" close=")"> 
        #{id}    
    </foreach>  
</delete>
```

- sql 标签 定义可重用的 SQL 片段
  - 提取可重用的 SQL 片段
  - 使用 include refid 引入 SQL 片段

# MyBatis 中缓存机制
- 一级缓存 sqlSession
- 二级缓存 sqlSessionFactory
- 第三方缓存 EhCache

## Mybatis 一级缓存
- local cache,本地缓存,他的作用域是在 sqlSession 级别
- 默认开启,无法关闭,可以清空

### 一级缓存机制
  - 第一次查询数据时,首先从数据库中获取数据,并将数据缓存值一级缓存中,它底层是一个 Map 集合
  - 再次来获取相同的数据时,先从一级缓存中获取数据,如果没有在一级缓存中获取到数据,再去数据库中获取,获取到数据之后将数据继续缓存到一级缓存中

###  一级缓存失效的几种情况
  - 不同的 sqlSession 对应不同的一级缓存
  - 同一个 sqlSession 但查询条件不同
  - 同一个 sqlSession 但两次查询中间有任意增删改操作
    - 执行增删改操作后,会默认清空缓存
  - 同一个 sqlSession 两次查询期间手动清空了缓存
    - `sqlSession.clearCache()`
  - 同一个 sqlSession 两次查询期间提交了事务
    - 提交事务也会默认清空缓存

## 二级缓存
- second level cache 全局作用域缓存,他的作用域为 sqlSessionFactory 或 nameSpace 级别
- 二级缓存默认是关闭的,需要手动开启才能生效

### 使用步骤
  1. 全局配置文件中开启二级缓存
 ```xml
<setting name="cacheEnabled" value="true"/>
 ```

  - 在 SQL 映射文件中使用 cache 配置缓存

     ```xml
          <cache></cache>
      ```

### 注意
  - 要使用二级缓存,POJO 需要实现序列化接口
  - 关闭 sqlSession 或提交 sqlSession 时,数据才会提交到二级缓存

### 二级缓存机制
  - 第一次获取数据,先从数据库中获取数据,会默认将数据缓存至一级缓存,当二级缓存开启,并关闭或提交 sqlSession 时将数据缓存至二级缓存
  - 再次获取数据,优先从二级缓存中获取数据,如二级缓存中没有该数据,再从一级缓存中获取,如未从一级缓存中获取数据,再从数据库中获取数据
  - 从数据库中获取后,再次缓存至一级缓存,后缓存至二级缓存

### 二级缓存的相关属性
  - eviction 缓存清除回收策略
    - FIFO 先进先出
    - LRU 最近最少使用,先回收
  - flushInterval 刷新间隔,单位毫秒
  - size 缓存的对象个数
  - readOnly 设置缓存只读
    - 只有查询时,才设置只读

## 第三方缓存
