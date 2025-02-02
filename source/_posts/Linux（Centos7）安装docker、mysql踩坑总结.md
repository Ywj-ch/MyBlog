---
title: Linux（Centos7）安装docker、mysql踩坑总结
date: 2025-01-12 21:05:12
tags: [docker, linux]
categories: 学习笔记
description: 记录了在 CentOS 7 上安装 Docker 和 MySQL 时遇到的问题及解决方案，帮助你更顺利地搭建环境，避免常见的坑。
cover: /img/Linux（Centos7）安装docker、mysql踩坑总结.png
swiper_index: 4
---

> 本文主要是记录了在 CentOS 7 上安装 Docker 和 MySQL 时遇到的一些问题，主要是由于镜像源未配置正确，导致无法顺利下载所需的依赖包。下面将介绍在安装过程中遇到的困难，并分享如何通过配置合适的镜像源来解决这些问题，从而顺利完成 Docker 和 MySQL 的安装，希望能够帮到有需要的人。

#### 一、安装准备

> 系统版本：CentOS 7
>
> 先安装 yum：

```java
yum install -y yum-utils device-mapper-persistent-data lvm2
```

> 执行之前先配置一下镜像源，输入以下命令进入配置文件：

```java
vim /etc/yum.repos.d/CentOS-Base.repo
```

> 再将 mirrorlist 注释掉然后将 baseurl 改为阿里云镜像，然后保存退出

![pic_8fcb7204.png](https://api.smain.cn/pics/pic_8fcb7204.png)

> 一定要将 mirrorlist 注释掉！不然还是会直接访问官方源导致下载失败！

> 输入下面的命令检验是否安装成功：

![pic_22eea1b9.png](https://api.smain.cn/pics/pic_22eea1b9.png)

> 当然不排除网络问题，可以先用 ping 命令测试一下网络是否连通：

![pic_869db52c.png](https://api.smain.cn/pics/pic_869db52c.png)

> 只要网络连通，并且配置文件修改无误就肯定能安装成功。

#### 二、安装 Docker

> 执行下列命令安装：

```java
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> 下面是 Docker 的一些常用命令：

```java
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps

# 列出本地 Docker 主机上已经下载或构建的所有镜像
docker images
```

> 可以看到刚刚安装好 docker 后是没有镜像的：

![pic_e644b8d0.png](https://api.smain.cn/pics/pic_e644b8d0.png)

#### 三、安装 MySQL

> 现在要使用 docker 安装 mysql 镜像，注意如果之前有在虚拟机上安装过 mysql 要先停掉，不然会端口冲突：

```java
systemctl stop mysqld
```

> 如果直接 docker pull mysql，多半会报错：

```java
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

> 所以还是先配置镜像，值得一提的是在写这篇文章的时候阿里云的镜像加速已经停用了，所以得找别的镜像。

> 创建一个 docker 的配置文件，然后设置镜像源：

```java
# 创建目录
mkdir -p /etc/docker

#创建文件
touch daemon.json

#进入编辑
vim /etc/docker/daemon.json
```

```java
{
  "registry-mirrors": [
    "https://gallery.ecr.aws/",
    "https://gitverse.ru/docs/artifactory/gitverse-registry/",
    "https://docker.lmirror.top",
    "https://atomhub.openatom.cn/"
  ]
}
```

> 这几个镜像目前测试是有效的，然后再 docker pull mysql
>
> 可以看到很快就下好了，再使用 docker images：

![pic_dd193e67.png](https://api.smain.cn/pics/pic_dd193e67.png)

> 成功拉取！最后下载 mysql：

```java
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  mysql
```

> 其中设置的用户名是 root ，密码是 123，如有需要自行更改。

![pic_8fda741d.png](https://api.smain.cn/pics/pic_8fda741d.png)

> 安装成功！来测试一下能否连接成功：
>
> 在 navicat 中新建连接，其中 Host 填你自己的虚拟机 IP 地址
>
> 使用 ifconfig 即可查看

![pic_c0b322a5.png](https://api.smain.cn/pics/pic_c0b322a5.png)

![pic_bbd51289.png](https://api.smain.cn/pics/pic_bbd51289.png)

#### 四、总结

> 最后有几点需要注意：
>
> 1、重新启动虚拟机后需要再次停用原来的 mysql 防止端口冲突
>
> 2、开启 dokcer 后 mysql 并没有一并被开启，需要输入 dokcer start mysql 开启
>
> 3、如果发现数据库连接不上了可以考虑关闭虚拟机的防火墙
>
> ```java
> # 查看防火墙状态
> firewall-cmd --state
>
> # 关闭防火墙
> systemctl stop firewalld
> ```
