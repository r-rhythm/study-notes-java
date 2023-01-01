# **第**1章 Java概述



## 1、Java的历史



Java目前已经是Java18了，但是我们学习的和用的仍然是Java8。



Java是95正式对外发布，96对外发布JDK开发工具。



Java诞生于SUN公司，现在归Oracle（甲骨文）。



Java之父：詹姆斯.高斯林。



## 2、Java的特点



面向对象、跨平台（JVM：Java虚拟机）、健壮、安全、支持分布式、支持多线程等。



## 3、搭建环境



JDK = JRE + 开发工具



JRE = JVM + 核心类库



JVM：Java虚拟机，只识别字节码格式数据，负责把字节码解释成CPU能识别的指令等。JVM有内存管理机制，有自己的垃圾回收（GC）机制。



JRE：Java运行环境。



JDK：Java开发工具包。



开发人员：安装JDK。



如果希望在任意目录下都可以使用javac,java等开发工具，最好配置环境变量path。



例如：我的JDK安装路径是：D:\ProgramFiles\Java\jdk1.8.0_333



那么需要在path环境变量中增加：D:\ProgramFiles\Java\jdk1.8.0_333\bin的新变量值。



或者新建一个环境变量：JAVA_HOME，它代表JDK的安装根目录，它的变量值是D:\ProgramFiles\Java\jdk1.8.0_333，然后path中刚刚JDK的环境变量值可以写成：%JAVA_HOME%\bin



## 4、HelloWorld程序



```java
/*
class是一个关键字，表示定义一个类
HelloWorld是一个类名，自己取的名，建议每一个单词的首字母大写
因为HelloWorld类中包含main方法，所以HelloWorld又称为主类

javac HelloWorld.java编译
java HelloWorld 运行
*/
class HelloWorld{
    /*
    main方法是Java程序的入口，它的格式是固定的，又称为主方法
    */
	public static void main(String[] args){
        /*
        这是一个输出语句，必须以;结尾
        ""表示字符串常量，这里表示输出字符串常量值 hello java
        */
		System.out.println("hello java");
	}
}
```



## 5、Java程序开发步骤和注意事项



（1）开发步骤



- 编写代码，保存为==**.java**==的源文件

- 编译源文件生成==**.class**==的字节码文件 

- - 编译命令：javac  源文件名.java

- - 为什么要编译？因为Java程序是在JVM（Java虚拟机）中运行的，JVM只能识别字节码数据。JVM的作用就是把字节码数据解释为对应CPU能识别的指令。

- 运行字节码文件 

- - 运行命令：java  主类名

- - 主类名是指包含main的class名称。



（2）注意事项



- 所有标点都必须==**英文半角输入**==

- Java是严格区分大小写，单词注意哪些字母是大写的，哪些是小写的，例如：String的首字母大写

- 成对的标点符号要成对，不能多也不能少，特殊是{}

- 注意代码的风格



```java
class 类名{
    public static void main(String[] args){
        代码1
        代码2
        嵌套的外层代码{
            嵌套的内层代码
        }
    }
}

注意{}对齐问题，和每一个层级的缩进问题，一个层级缩进一个Tab键
看到{才缩进
```



- 建议主类名和xx.java源文件名同名，包括大小写形式也一致，便于维护

- 中文要注意编码问题