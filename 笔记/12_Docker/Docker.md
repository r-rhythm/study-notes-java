# Docker
# 概述
> [!info] 什么是Docker
> - 保持代码运行环境一致的一种容器技术
> - Docker能够保证运行代码的环境一致
> - Docker的主要目标,一次封装,随处运行"*Build,Ship and Run Any App,Anywhere*"
> - 将代码和运行环境一起搬到其他环境运行,保证本地的运行结果与测试,生产的运行结果一致

# 基本概念
  1. 在客户端(*Client*)使用命令 docker pull 镜像名称,它会到注册中心(远程仓库)找到对应的镜像进行下载
  2. 从注册中心下载镜像(*Image*)后,保存到docker服务中
  3. 镜像通过容器(*Containers*)的方式进行使用
  4. 镜像相当于类,容器相当于对象,容器是镜像的实例,一个镜像可以创建多个容器
  5. Dockers守护进程,负责支撑Docker容器的运行及镜像的管理

# 安装使用
1. 安装必要的系统工具
```shell
yum -y install gcc

yum -y install gcc-c++

yum install -y yum-utils device-mapper-persistent-data lvm2
```
2. 添加软件源信息
```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
3. 更新并安装Docker-CE
```shell
yum makecache fast

yum -y install docker-ce
```
4. 开启Docker服务
```shell
systemctl start docker
```
5. 测试是否安装成功
```shell
docker -v

docker info # 查看详情
```
最后修改docker镜像
```shell
vim  /etc/docker/daemon.json

# 添加如下内容
{ 
  "registry-mirrors":["https://registry.docker-cn.com"] 
}
```


# 常用命令

## 镜像命令
- 查看当前的镜像 docker images
- 根据关键字搜索镜像 docker search + 镜像关键字
- 下载镜像 docker pull + 搜索到的镜像名称:版本
- 强制删除镜像 docker rmi -f + 镜像的id  (*remove image*)

## 容器命令
- 创建于启动容器 docker run
	- -i 运行容器
	- -d 守护进程(后台运行)
	- --name 为容器取别名
	- -v 目录挂载
	- -t 交互终端
	- -e 环境变量
	- -p 端口映射
	```shell
	# 后台启动(最常用)
	docker run -di --name=mycentos10 centos:7
	# 进入到容器内部
	docker exec -it mycentos10 /bin/bash
	```
- docker start 容器名称
- docker stop 容器名称
- 查看容器
	- 查看所有容器 docker ps -a
	- 查看正在运行的容器 docker ps
- 删除容器 docker rm -f + 容器名称或id
- docker logs -f 容器名称 查看日志

目录挂载:将Linux的目录与容器连接起来,Linux/容器执行修改或删除等都会影响另一方,相当于是同步了这个目录,这个目录始终保持一致,这样做的好处是修改Linux的内容它会自动同步到容器中去`docker run -di --name=mycentos3 -v /usr/local:/usr/local centos:7`


# 迁移备份
- docker commit 将容器保存为镜像
- docker save -o 文件名 镜像名 将镜像备份为tar文件
- docker load -i 文件名 将 tar文件恢复为容器


# 制作 Docker 镜像
准备工作
- Dockerfile 文件,里面指定 jdk 版本,缓冲区位置及 jar 包名称
- jar 包


---
## Dockers私有仓库
## 搭建私有仓库
1. 拉取私有仓库镜像`docker pull registry`
2. 启动私有仓库容器
3. 配置受信任地址,在deamon.json文件中添加`"insecure-registries":["192.168.6.200:5000"]`
4. 进入[http://192.168.6.200:5000/v2/_catalog](http://192.168.6.200:5000/v2/_catalog) 查看效果
