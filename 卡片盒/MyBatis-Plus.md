# 概述

- [MyBatis-Plus (baomidou.com)](https://baomidou.com/)

> MyBatis-Plus（简称 MP）是一个 [[MyBatis]] 的增强工具，在 [[MyBatis]] 的基础上只做增强不做改变，为简化开发、提高效率而生。

# 字段映射注解

- @TableName 指定这个类在数据库中对应的表名
- @TableField 指定类中属性对应的表中个字段
  - exist 指定表中是否可以必须存在对应字段
- @TableLogic 标记该属性对应的字段为判断逻辑删除 ~~标注逻辑删除的字段属性~~
- @TableId 主键注解
  - 这个注解通常会指定 IdType **主键策略**

## IdType 主键策略
 - AUTO 使用数据库自增的注解,要求数据库支持自增(*auto_increment*)
 - INPUT 手动指定主键,MyBatisPlus 不会自动生成
 - ASSIGN_ID(**默认**) 如果手动指定了 id,使用指定的 id,否则使用雪花算法随机生成一个 19 位的 id
 - ASSIGN_UUID 如果手动指定了 id,使用指定的 id,否则使用 UUID 随机生成字符串类型 id(*使用这个策略需要把类中属性和表结构的类型改为 String|varchar*)

 [详情参阅 MP 官网](https://baomidou.com/pages/223848/)

# MyBatis-Plus 中的两种分页方法
## MyBatis-Plus 分页
- 创建一个 MyBatisPlusConfig 配置类
- 使用@Configuration 标记他为一个配置类
- **注意,这个拦截器方法必须用使用注解装配,这里使用的是@Bean**
- 装配一个 MybatisPlusInterceptor,让他返回一个分页拦截器

   ```java
    package com.atguigu.system.config;
    
    import com.baomidou.mybatisplus.annotation.DbType;
    import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
    import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
    import org.springframework.boot.SpringBootConfiguration;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.annotation.Bean;
    import org.springframework.transaction.annotation.EnableTransactionManagement;
    
    /**
     * @author: Wei
     * @date: 2022/9/16 15:58
     *
     * MyBatis-Plus配置类
     */
    
    @SpringBootConfiguration
    @EnableTransactionManagement
    public class MyBatisPlusConfig {
    
        @Bean // 把这个拦截器装配到Spring
        public MybatisPlusInterceptor addPaginationInnerInterceptor(){
        
            MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
            
            // 向MyBatis-Plus过滤器链中添加分页拦截器
            interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
            
            return interceptor;
        }
    
    }
    ```

- 在 SysRoleMapper 内写一个分页方法
   ```java
    package com.atguigu.system.mapper;
    
    import com.atguigu.model.system.SysRole;
    import com.atguigu.model.vo.SysRoleQueryVo;
    import com.baomidou.mybatisplus.core.mapper.BaseMapper;
    import com.baomidou.mybatisplus.core.metadata.IPage;
    import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
    import org.apache.ibatis.annotations.Param;
    
    /**
     * @author: Wei
     * @date: 2022/9/16 9:26
     */
    
    public interface SysRoleMapper extends BaseMapper<SysRole> {
        /**
         * 在这个项目中将查询条件封装成了一个实体类<class>SysRoleQueryVo</class>,
         * 这样做的好处是提高可扩展性及减少重复的mapper和sql代码,
         * 重复使用一个查询条件实体类即可通过任意条件查询,大大提高了代码的复用性
         *
         * @param page
         * @param queryVo 手动封装的条件查询类
         * @return
         */
        IPage<SysRole> findAll(Page<SysRole> page,
                               @Param("vo") SysRoleQueryVo queryVo);
        
    }
    ```

- resources 内新建映射文件书写 Sql 语句
   ```xml
    <mapper namespace="com.atguigu.system.mapper.SysRoleMapper">
    
        <sql id="selectRole">
            select id,
                   role_name,
                   role_code,
                   description,
                   create_time,
                   update_time,
                   is_deleted
            from `guigu-auth`.sys_role
        </sql>
    
        <select id="findAll" resultType="sysRole">
            <include refid="selectRole"></include>
            <where>
                <if test="vo.roleName != null and vo.roleName != ''">
                    and role_name like "%"#{vo.roleName}"%" <!-- 这里是like模糊查询,不要错写成=等于了-->
                </if>
                and is_deleted =0
            </where>
            order by id desc
        </select>
    
    </mapper>
    ```

- controller
  ```java
  /**
       * 使用自定义的条件查询实体类<class>SysRoleQueryVo</class>
       * 实现分页及条件高级查询
       *
       * @param current 当前页码
       * @param size    每页显示条数
       * @param queryVo 手动封装的查询条件对象
       * @return
       */
      @ApiOperation("分页及条件查询")
      @GetMapping("/{current}/{size}")
      public Result getRsByRsVo(
              @ApiParam(name = "current", value = "当前页", required = true)
              @PathVariable Long current,
              
              @ApiParam(name = "size", value = "每页显示条数", required = true)
              @PathVariable Long size,
              
              @ApiParam(name = "queryVo", value = "查询条件", required = false)
                      SysRoleQueryVo queryVo) {
          
          
          Page<SysRole> sysRolePage = new Page<>(current, size);
          
          IPage<SysRole> result = sysRoleService.getAllByRsVo(sysRolePage, queryVo);
      
          System.out.println("result = " + result);
          
          return Result.ok(result);
  
      }
  ```

## MyBatis-Plus 分页查询最简化
- 在配置类中==装配一个分页插件 ==PaginationInterceptor
- 创建一个 Page\<T>对象,对象内传递当前页和每页记录数
- 调用 MP 的 selectPage 方法,传入 Page 对象和条件
- 查询后的结果会封装在这个 Page 对象中,直接调用这个 Page 对象即可取到

# MP[[乐观锁]]
- 在表中添加一个字段作为版本号,在属性中添加一个对应的属性
- MP 中使用 @Version 注解实现乐观锁,在对应的属性上添加这个注解
- 在配置类配置乐观锁的插件 OptimisticLockerInterceptor

# 查询
## 带条件的查询

- 设置查询条件
  - 创建 QueryWrapper 对象
    - `new QueryWrapper`
    - 查询后的数据都是封装在这个对象内的

  - 设置查询条件

    - gt 大于
    - lt 小于
    - eq 等于
    - 模糊查询
      - like 左右两边加%
      - likeLeft 左边加 %
      - likeRight 右边加 %
    - 指定需要查询的字段
      - `queryWrapper.select(column)`
  - 测试

    ```java
    /**
         * 根据条件查询
         */
        @Test
        public void testQueryWrapper(){
         
            //创建Wrapper对象
            QueryWrapper<SysRole> sysRoleQueryWrapper = new QueryWrapper<>();
            
            /*
            * 设置查询条件
            *   查询role_name 以管理员结尾的所有字段
            * */
            sysRoleQueryWrapper.likeLeft("role_name","管理员");
        
            /*
            * SELECT 
            *   id,role_name,role_code,description,create_time,update_time,is_deleted 
            * FROM 
            *   sys_role WHERE is_deleted=0 
            * AND 
            *   (role_name LIKE '管理员%')
            * */
            List<SysRole> sysRoles = sysRoleMapper.selectList(sysRoleQueryWrapper);
            for (SysRole sysRole : sysRoles) {
                System.out.println("sysRole = " + sysRole);
            }
    
        }
    ```

- 设置条件更新
  - 创建 UpdateWrapper 对象 `new UpdateWrapper<>()`
    - `updateWrapper.set().条件|eq 等()`
    - 示例

      ```java
      /**
           * 测试根据条件更改
           */
          @Test
          public void testUpdateByWrapper(){
          
              UpdateWrapper<SysRole> wrapper = new UpdateWrapper<>();
              
              
              /*
              *  UPDATE sys_role SET description=? WHERE is_deleted=0 AND (id = ?)
              *   
              *   将Id为10的字段description更改为编剧
              * 
              * */
              wrapper.set("description","编剧").eq("id",10L);
              
              sysRoleMapper.update(null,wrapper);
          
          }
      ```

## 条件查询优化 Lambda 
> 使用标准的条件查询,必须要求查询条件与表中字段相同,使用Lambda查询则无需官族字段

```java
@Test  
public void lambdaSelect(){  
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();  
    wrapper.eq(User::getAge,18); // 先封装条件,这样就无需关注表中字段,因为表中字段在实体类中已经建立映射关系了  
    List<User> users = userMapper.selectList(wrapper);  
    users.forEach(System.out::println);  
}
```


# MyBatis-Plus对Service层的封装
## Service 接口

* 创建Service接口,继承`IService<T>`并指定泛型
* IService这是Mybatis-Plus提供的默认Service接口。

  ```java
  package com.atguigu.system.service;
  
  import com.atguigu.model.system.SysRole;
  import com.atguigu.model.vo.SysRoleQueryVo;
  import com.baomidou.mybatisplus.core.metadata.IPage;
  import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
  import com.baomidou.mybatisplus.extension.service.IService;
  import org.apache.ibatis.annotations.Param;
  
  /**
   * @author: Wei
   * @date: 2022/9/16 13:49
   */
  public interface SysRoleService extends IService<SysRole> {}
  ```


  ## Service接口实现
  * 继承MP提供的实现类`IService<M,T>`实现自己的Service接口
  * ServiceImpl这是Mybatis-Plus提供的默认Service接口实现。

   ```java
    /**
     * @author: Wei
     * @date: 2022/9/16 13:50
     *
     * 继承MyBatis-Plus提供的实现类,实现我们自己的接口
     *
     * 泛型指定为我们自定的Mapper和自定的实体类
     */
    @Service
    public class SysRoleServiceImpl extends ServiceImpl<SysRoleMapper, SysRole> implements SysRoleService {
        
    }
    ```

* 测试
  * 查询`sysService.list()` 查询所有
  * 添加`sysService.save(T)`添加
  * 更新`sysService.updateByID()`
  * 删除`sysService.removeById()`

# 补充内容
## 常见
1. @Mapper 注解记得加
2. MP 内建了连接池 Hikari
3. JavaScript 处理数组类型只能处理 16 (+- 2^53)位,超出后会有精度问题,所以使用雪花算法生成的 id 需要用 String 类型来保存
4. Service 中要调用 Mapper 不需要再手动注入一遍,在接口泛型传入 Mapper 的时候MP就帮我们注入了 baseMapper,直接调用即可

## 绑定异常
绑定异常可能出现的原因
1. 没有遵循三个一致
2. Mapper 的 XML 文件加载问题,使用 [[MyBatis-Plus]] 代码生成器生成的 XML 文件是存放在 src-main-java 路径下的,但 maven 项目默认只会编译这个路径下的 Java 文件,这就会导致这个 XML 文件无法编译与加载[[MyBatis#常见问题]]

绑定异常解决
- 通过配置 maven 加载资源解决
	1.  在 pom 文件中添加配置
	2. 在 properties | yml 配置文件中添加配置
