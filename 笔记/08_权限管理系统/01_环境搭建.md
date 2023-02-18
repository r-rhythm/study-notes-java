## 介绍

### 后端架构

* SpringBoot
* MyBatis-Plus
* Spring Security

### 前端架构

* Node.js
* NPM
* Vue
* ElementUI
* Axios

### 核心技术

* 基础框架 SpringBoot
* 数据缓存:Redis
* 数据库:MySql
* 权限控制:SpringSecurity
* 日志记录:AOP
* 前端...

### 项目模块

最终服务器端架构模块

guigu-auth-parent：根目录，管理子模块

​	 common：公共类父模块

​		common-log：系统操作日志模块

​		common-util：核心工具类

​		service-util：service模块工具类

​		spring-security：spring-security业务模块

​	model：实体类模块

​	service-system：系统权限模块



## 环境搭建

### 搭建项目结构

#### 创建父工程guigu-auth-parent

* GAV Version 改为 1.0 去掉snapshot
* 删除src目录

#### 创建父模块common

* 继承父工程
* 删除src

#### 创建common-util

* 继承common
* 建包`com.atguigu` 导入工具类common

#### 创建service-util

* 继承common

#### 搭建实体类模块model

* 继承父工程
* 创建`com.atguigu`包
* 引入资料内的实体类model

#### 搭建项目模块service-system

* 继承父工程



### 配置依赖关系

* 在父工程进行统一的jar包管理

  * ```xml
    <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.3.6.RELEASE</version>
        </parent>
        <properties>
            <java.version>1.8</java.version>
            <alibaba.version>2.2.0.RELEASE</alibaba.version>
            <mybatis-plus.version>3.4.1</mybatis-plus.version>
            <mysql.version>8.0.23</mysql.version>
            <knife4j.version>2.0.8</knife4j.version>
            <jwt.version>0.7.0</jwt.version>
            <fastjson.version>1.2.29</fastjson.version>
        </properties>
    
        <!--配置dependencyManagement锁定依赖的版本-->
        <dependencyManagement>
            <dependencies>
                <!--mybatis-plus 持久层-->
                <dependency>
                    <groupId>com.baomidou</groupId>
                    <artifactId>mybatis-plus-boot-starter</artifactId>
                    <version>${mybatis-plus.version}</version>
                </dependency>
                <!--mysql-->
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>${mysql.version}</version>
                </dependency>
                <!--knife4j-->
                <dependency>
                    <groupId>com.github.xiaoymin</groupId>
                    <artifactId>knife4j-spring-boot-starter</artifactId>
                    <version>${knife4j.version}</version>
                </dependency>
                <!--jjwt-->
                <dependency>
                    <groupId>io.jsonwebtoken</groupId>
                    <artifactId>jjwt</artifactId>
                    <version>${jwt.version}</version>
                </dependency>
                <!--fastjson-->
                <dependency>
                    <groupId>com.alibaba</groupId>
                    <artifactId>fastjson</artifactId>
                    <version>${fastjson.version}</version>
                </dependency>
            </dependencies>
        </dependencyManagement>
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.1</version>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    ```

* common-util

  * ```xml
    <packaging>jar</packaging>
    
        <dependencies>
            <dependency>
                <groupId>com.atguigu</groupId>
                <artifactId>model</artifactId>
                <version>1.0</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>io.jsonwebtoken</groupId>
                <artifactId>jjwt</artifactId>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
            </dependency>
        </dependencies>
    ```

* service-util模块

  * ```xml
    <dependencies>
            <dependency>
                <groupId>com.atguigu</groupId>
                <artifactId>common-util</artifactId>
                <version>1.0</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
            </dependency>
            <!--mysql-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
        </dependencies>
    ```

* model模块

  * ```xml
    <dependencies>
            <!--lombok用来简化实体类-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </dependency>
            <dependency>
                <groupId>com.github.xiaoymin</groupId>
                <artifactId>knife4j-spring-boot-starter</artifactId>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    ```

* service-system

  * ```xml
    <packaging>jar</packaging>
    
        <dependencies>
            <dependency>
                <groupId>com.atguigu</groupId>
                <artifactId>service-util</artifactId>
                <version>1.0</version>
            </dependency>
            
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
    
        <build>
            <finalName>${project.artifactId}</finalName>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    ```



