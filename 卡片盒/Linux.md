# 补充
* tail -f ... -n 50 滚动的查看文件最后 50 行信息*
* mkdir -p /a/bb/cc创建多层级目录
* /etc/profile Linux 的核心配置文件,环境变量等都存放在这个文件中

# 常用基本命令


### VI&VIM编辑器

`yum -y install vim`安装VIM

#### 一般模式

* yy 复制光标当前一行
* p 粘贴
* u 撤销
* dd 删除当前行
* x delete
* X BackSpace
* yw 复制一个词
* dw 删除一个词
* ^ 移动到行头
* $ 移动到行尾

#### 编辑模式

* i 插入
* a 添加
* o 下一行输入

#### 命令模式

* :w 保存
* :q 退出
* :! 强制退出
* :%s/old/new 使用新则字符替换旧的字符
* :nohl 关闭高亮
* :set nu 显示行号
* :set nonu 关闭行号
* ZZ 保存文件退出



### 网络配置

* ifconfig 显示所有网络接口的配置信息

* 修改IP地址

  * `vim /etc/sysconfig/network-scripts/ifcfg-ens33`

  * `BOOTPROTO=static` \#IP的配置方法[ none | static | bootp | dhcp ]（引导时不使用协议|静态分配IP|BOOTP协议|DHCP协议)

  * `ONBOOT=yes`

  * ```shell
    #IP地址
    IPADDR=192.168.6.100   
    #网关  
    GATEWAY=192.168.6.2      
    #域名解析器
    DNS1=192.168.6.2
    ```



### 配置主机名

* hostname 查看当前服务器的主机名称
  * `vim /etc/hostname` 修改当前主机名称
  * `vim /etc/hosts` 为IP地址映射主机名称



### 服务管理类命令

* 开启及关闭服务

  * ```shell
    systemctl  	start		服务名		（功能描述：开启服务）
    systemctl  	stop		服务名		（功能描述：关闭服务）
    systemctl   restart	 	服务名		（功能描述：重新启动服务）
    systemctl   status	 	服务名		（功能描述：查看服务状态）
    systemctl  --type  service		   （功能描述：查看正在运行的服务）
    ```

* 设置后台服务自启

  * ```shell
    systemctl  list-unit-files   	   （功能描述：查看所有服务器自启配置）
    systemctl  disable 		服务名   	（功能描述：关掉指定服务的自动启动）
    systemctl  enable  		服务名  	（功能描述：开启指定服务的自动启动）
    systemctl  is-enabled 	服务名		（功能描述：查看服务开机启动状态）
    ```

* 关机重启命令

  * > 在[[Linux]]领域内大多用在服务器上，很少遇到关机的操作。毕竟服务器上跑一个服务是永无止境的，除非特殊情况下，不得已才会关机。
    >
    > **正确的关机流程为**：sync > shutdown > reboot >poweroff

  * ```shell
    sync  			（功能描述：将数据由内存同步到硬盘中）
    poweroff		（功能描述：关闭系统，等同于shutdown -h now）
    reboot 			（功能描述：就是重启，等同于 shutdown -r now）
    shutdown [-h|-r] [min] 在几分钟后关机或重启
    ```

### 帮助类命令

* 查看命令手册manual

  * ```shell
    man [命令或配置文件]		（功能描述：获得帮助信息)
    
    信息					功能
    NAME			命令的名称和单行描述
    SYNOPSIS		怎样使用命令
    DESCRIPTION		命令功能的深入讨论
    EXAMPLES  		怎样使用命令的例子
    SEE ALSO		相关主题（通常是手册页）
    ```

* help

  * ```shell
    help 命令	（功能描述：获得shell内置命令的帮助信息）
    ```

* 其他快捷键`tab ↑ ↓`等



### 文件目录类

* pwd 打印当前目录

  * cd[参数] 切换目录

  * `cd~`返回Home目录
* ls[选项] [目录或是文件] 列出目录内容
* mkdir  创建目录

  * `mkdir -p /dir1/dir2` 创建多层目录
* rmdir  删除一个**空的**目录
* cp [选项] source dest  复制source文件到dest  
* mv a dest 移动a文件到dest
* `rm [选项] deleteFile`递归删除目录中所有内容

  * `-f`不提示进行确认
* cat 查看文件内容
* less 分屏查看
* echo 输出内容到控制台
* head 显示头十行
* tail 显示尾十行
* \> 覆盖
* \>> 追加
* ln 软连接,类似快捷方式,或给文件起别名
* history 查看已执行过的历史命令



### 时间日期类

* date 
* cal 查看日历



### 用户管理类命令

#### 用户

* useradd 添加新用户
  * useradd -g 组名 用户名 
* `usermod -g 组名 用户名` 修改用户组
* passwd 为账户设置密码
* id 查看用户是否存在
* su 切换用户
  * `su - 用户名称`切换用户并获得该用户的环境变量及执行权限
* who 查看登录用户信息
  * whoami 显示自身用户名称
  * who am i 显示登录用户的名称

#### 组

* groupadd 创建组
* groupdel 删除组
* `groupmod -n 新组名 老组名` 更改组名

#### 文件权限类

* 权限属性

  * 0 类型

  * 1~3 属主权限

  * 4~5 属组权限

  * 7~9 other权限

  * r | w | x | ==> 读 | 写 | 执行

* 权限变更

  * `chmod [{ugoa}{+-=}{rwx}] 文件或目录`
  * **`chmod 421 [文件或目录]`**
    * 该文件属主只读,属组只写,其他用户只执行
    * **4=r , 2=w , 1=x**
    * rwx = 4+2+1 =7

* 改变属主

  * `chown 目标属主 文件名`

* 改变属组

  * `chgrp 目标组 文件名`

#### 搜索查找类

* find 
  * `find /目录 -name "name/user/size"`
* grep 过滤查找,通常配合管道符" | "使用,它表示将已处理的数据交给后面的命令处理
  * `ll | grep a`



#### 压缩解压缩类

* tar [选项] XXX.tar.gz

  * -z 打包的同时压缩
  * -c 产生.tar包文件
  * -v 显示详细星系
  * -f 指定压缩后的文件名
  * -x 解包.tar文件
  * `tar -zcvf ab.tar.gz a b`将a和b打包为ab.tar.gz
  * `tar -zxvf ab.tar.gz -C dir/`将ab解压到指定路径

  

  

  #### 磁盘分区类

  *  df -h 查看磁盘空间的空余情况
  * mount/unmount 挂载/卸载
    * `mount -t iso9660 /dev/cdrom /mnt/cdrom/`cdrom需要自行创建
    * `umount`



#### 进程线程类

* ps 查询进程状态
  * ps -aux 查看系统中所有线程
  * ps -ef 可以查看子父进程之间的关系
  * 配合管道符 | grep XX筛选出要查询的进程
* kill
  * kill[选项] 进程号
  * killall 进程名

#### cront 定时任务设置

* cronttab[选项]

  * -e 编辑定时任务
  * -l 查询crontab任务
  * -r 删除所有crontab任务

* \* \* \* \* *

   | 第一个“*” | 一小时当中的第几分钟 | 0-59                    |
    | --------- | -------------------- | ----------------------- |
    | 第二个“*” | 一天当中的第几小时   | 0-23                    |
    | 第三个“*” | 一个月当中的第几天   | 1-31                    |
    | 第四个“*” | 一年当中的第几月     | 1-12                    |
    | 第五个"*" | 一周当中的星期几     | 0-7（0和7都代表星期日） |

  * `45 22 * * *命令` 在每天22点45分执行命令

### 软件包管理

#### RPM

###### RPM查询命令
* rpm -qa 查询所有安装的rpm包

###### RPM卸载命令
* rmp -e RPM软件包 卸载软件包

###### RPM安装命令
* rmp -ivh 全包名


#### YUM仓库
* `yum[选项] [参数]`

* 选项

  * -y 对所有提问都回答yes

* 参数

  * | 参数         | 功能                          |
    | ------------ | ----------------------------- |
    | install      | 安装rpm软件包                 |
    | update       | 更新rpm软件包                 |
    | check-update | 检查是否有可用的更新rpm软件包 |
    | remove       | 删除指定的rpm软件包           |
    | list         | 显示软件包信息                |
    | clean        | 清理yum过期的缓存             |
    | deplist      | 显示yum软件包的所有依赖关系   |

* 示例

  * `yum -y update firefox` 更新firefox

# bug
- 设置 ens33 后该配置不生效,外部仍然无法通过静态 IP 连接到虚拟机.将 NetworkManager 的开机自启禁用,让 Linux 在开机后只是用 ens33 配置的网络设置即可解决 #bug