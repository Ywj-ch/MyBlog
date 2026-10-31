---
title: Spring Boot + Vue 前后端分离项目上线实记
date: 2025-02-6 18:29:31
tags: [云服务]
categories: 学习笔记
description: 记录了一个前后端分离项目的完整部署流程。
cover: /img/项目上线.webp
swiper_index: 5
---

### 一、前言

> 本文记录了一个前后端分离项目的完整部署流程。许多同学在本地开发完项目后，缺乏将其部署到远程服务器的实战经验。因此，我将以一个实际项目为例，详细展示从环境准备到最终上线的全过程，希望对你有所帮助！

### 二、服务器环境准备

#### 2.1、配置服务器

> 首先准备一个服务器，我这里用的是阿里云 ：

![pic_1730b086.png](/images/2025/springboot-vue-project/1.webp)

![pic_93189640.png](/images/2025/springboot-vue-project/2.webp)

> 第一次购买一般都会有优惠，而且对于一般的单体项目而言 2 核 2G 的配置也够用了，当然也有 3 个月的试用版本。

> 完成之后进入控制台第一件事就是去你左侧边栏的安全组里面开放端口，这样才能从远程访问。

![pic_78a16574.png](/images/2025/springboot-vue-project/3.webp)

![pic_ef901e7b.png](/images/2025/springboot-vue-project/4.webp)

> 端口范围根据需求开放，一般是 3306 mysql、80 nginx 代理、13103 这里是宝塔面板的端口，什么是宝塔面板后面会讲到，下面 3 个端口是默认初始化好的。

> 然后进入实例面板重新设置一下你的服务器密码：

![pic_607c2eda.png](/images/2025/springboot-vue-project/5.webp)

> 修改完成之后就可以试着连接你的服务器了，我这里使用的是 Xshell：

![pic_a2720479.png](/images/2025/springboot-vue-project/6.webp)

> 然后输入你刚才重新设置的密码并点击连接：

![pic_38be3af4.png](/images/2025/springboot-vue-project/7.webp)

![pic_82b1a136.png](/images/2025/springboot-vue-project/8.png)

#### 2.2、安装所需环境

> 成功连接好了服务器之后根据你的项目分析一下所要安装的环境
>
> - jdk
> - Mysql、Redis
> - Nginx

> 这里我们使用宝塔面板来管理我们的 Linux 服务器：

![pic_b3dc1254.png](/images/2025/springboot-vue-project/9.webp)

> 直接复制代码贴到 Xshell 执行即可：

```java
if [ -f /usr/bin/curl ];then curl -sSO https://download.bt.cn/install/install_panel.sh;else wget -O install_panel.sh https://download.bt.cn/install/install_panel.sh;fi;bash install_panel.sh ed8484bec
```

> 安装好后会出现外网地址和用户名和密码，打开外网地址后就能看到宝塔登录界面。

![pic_b3c89e3a.png](/images/2025/springboot-vue-project/10.webp)

![pic_b83c902c.png](/images/2025/springboot-vue-project/11.webp)

> 如果访问不通就是你没有放行端口，进入阿里云安全组里面去放行对应的端口即可。

> 进入控制面板后直接在右下角的软件商店中安装所需环境：

![pic_25f14775.png](/images/2025/springboot-vue-project/12.webp)

> 点开首页就可以清楚的看到我们服务器的整体情况了：

![pic_0dd8d85e.png](/images/2025/springboot-vue-project/13.webp)

### 三、后端部署

#### 3.1、项目打包

> 基于 Spring Boot + Vue 的前后端分离项目，主要有两种部署方式：

> 1. 前后端一起部署
>
> 这种方式将 Vue 项目打包后的 `dist` 目录放入 Spring Boot 项目的资源目录（如 `src/main/resources/static`），然后直接部署 Spring Boot 项目。实质上，前端文件仍然由后端的 Tomcat 服务器提供，并不是真正的前后端分离。
>
> - 优点：部署简单，无需额外的跨域配置。
> - 缺点：如果项目较大，会给 Tomcat 服务器带来较大压力，影响响应速度，且不利于前后端分工协作。

> 2. 前后端单独部署（推荐）
>
> 在这种方式下，Spring Boot 项目和 Vue 项目分别独立部署：
>
> - 后端：Spring Boot 项目打包后运行，使用内嵌 Tomcat 提供 API 服务。
> - 前端：Vue 项目打包后交由 Nginx 运行。
> - 请求处理：Nginx 负责端口转发，解决跨域问题，使前端能正常访问后端接口。
>
> 相比第一种方式，这种方式稍微复杂一些，但更符合前后端分离架构，能够减少后端服务器压力，提高系统性能。

> 本文将介绍 第二种方式 的部署流程，并通过 宝塔面板 进行可视化部署。正式开始之前，需要先确保环境就绪。

> 在对后端项目进行打包之前还需要做一些修改，如果你的配置文件不是动态加载的话那就去修改一下你的数据库连接配置：

![pic_45a5d47b.png](/images/2025/springboot-vue-project/14.webp)

> 然后将可以进行打包部署了，在你父模块（如果是聚合工程的话）的 lifecycle 下先运行 clean 再运行 package 进行打包：

![pic_991cbf5d.png](/images/2025/springboot-vue-project/15.png)

> 打包完成后会在你的 target 目录下生成对应的.jar 包，然后将其上传到服务器上。

#### 3.2、创建数据库

> 在宝塔面板中创建数据库并导入 sql 脚本：

![pic_72876713.png](/images/2025/springboot-vue-project/16.webp)

![pic_dedc1972.png](/images/2025/springboot-vue-project/17.webp)

> 在你的数据库根目录下启动终端运行下面一段代码即可将当前数据库的数据导出为.sql 脚本：

```java
mysqldump -u username -p database_name > /path/to/backup.sql
```

> - `-u username`：指定数据库用户名。
> - `-p`：提示输入数据库密码。
> - `database_name`：要导出的数据库名。
> - `/path/to/backup.sql`：备份文件保存的路径。

> 运行后会提示你输入数据库密码，完成后即可成功导出。

![pic_adfe5b50.png](/images/2025/springboot-vue-project/18.webp)

#### 3.3、测试启动

> 数据库建好后就可以去启动你的后端服务看一下是否成功了

```java
nohup java -jar 你的项目.jar --server.port=8080 > app.log 2>&1 &
```

> - `nohup` 是一个 Linux 命令，意思是 “no hangup”，它用来在退出终端后继续运行命令。通常，命令在终端关闭时会终止，但使用 `nohup` 后，即使关闭终端，程序依然会继续运行。
> - `--server.port=8080`：指定应用程序的 HTTP 服务端口为 `8080`，默认情况下 Spring Boot 应用会监听 `8080` 端口，但可以通过该参数修改端口。
> - `> app.log`：  
>   这是将标准输出（stdout）重定向到文件 `app.log` 中。
>
>   `>` 是重定向符号，它会把命令的输出写入到指定文件中，如果文件不存在会自动创建。`app.log` 是输出日志文件的名称，这里创建一下日志文件方便调试。
>
> - `&`：这个符号将命令放到后台运行。
>
>   在命令末尾加 `&` 可以使该命令在后台运行，不会阻塞当前终端。你可以继续在终端中进行其他操作。

![pic_87b54246.png](/images/2025/springboot-vue-project/19.png)

> ok 没有问题，下面进入前端部署。

### 四、前端部署

#### 4.1、项目打包

> 在前端终端输入以下命令进行打包：

```java
npm run build
```

> 具体情况可以结合你自己 package.json 文件里面的设置执行：

![pic_f4902b02.png](/images/2025/springboot-vue-project/20.webp)

> 打包完成后会在你项目的第一级目录下生成 dist 文件夹：

![pic_0c1a0aea.png](/images/2025/springboot-vue-project/21.webp)

> 然后将这个文件夹上传到服务器，我个人的习惯是和后端 jar 包放在一起：

![pic_2998d1e6.png](/images/2025/springboot-vue-project/22.png)

#### 4.2、nginx 配置

> 为了让前端和后端的请求能够正确地通过 Nginx 进行转发，我们需要配置 Nginx 实现反向代理。具体配置如下：

```java
# 处理 RyVending 项目的前端请求
      server {
        listen       80;
        server_name  你的服务器ip地址;
		    charset utf-8;
        location / {
            root   /www/wwwroot/RyVending/dist（你的dist文件的目录）;
			      try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }

		location /prod-api/（你自己的路由） {
      	    proxy_set_header Host $http_host;
      		proxy_set_header X-Real-IP $remote_addr;
      		proxy_set_header REMOTE-HOST $remote_addr;
      		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      		proxy_pass http://localhost:8080/;
		}

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

> - `location /`：这个 `location` 块用于处理所有指向根路径 `/` 的请求，通常是前端页面的访问。
> - `root /www/wwwroot/RyVending/dist;`：指定 Vue 前端打包后的静态文件所在的目录，这里是 `dist` 目录。`dist` 目录是 Vue 打包后的输出目录，包含所有 HTML、CSS、JS 文件。
> - `try_files $uri $uri/ /index.html;`：这是为了支持 Vue 的前端路由。如果用户直接刷新页面或访问子路由，Nginx 会先尝试查找具体的文件或目录。如果找不到，则返回 `index.html`，从而让 Vue 路由控制页面显示。

> - `location /prod-api/`：这个块用于处理所有以 `/prod-api/` 开头的请求，将其转发到后端的 Spring Boot 应用。这里的 `/prod-api/` 是你自定义的后端 API 路径前缀，可以根据你的需要修改。
> - `proxy_set_header Host $http_host;`：将客户端请求的 `Host` 头部转发给后端服务器，这样后端应用就能知道真实的主机名。
> - `proxy_set_header X-Real-IP $remote_addr;`：将客户端的真实 IP 地址传递给后端应用，通常用于日志记录或安全检查。
> - `proxy_set_header REMOTE-HOST $remote_addr;`：这个设置也用于传递客户端的 IP 地址（与 `X-Real-IP` 类似）。
> - `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`：这个设置会将客户端的真实 IP 添加到 `X-Forwarded-For` 头中，它可以帮助后端应用识别经过反向代理的真实请求来源。
> - `proxy_pass http://localhost:8080/;`：这是反向代理指令，表示将所有请求转发到本地的 `8080` 端口（即后端的 Spring Boot 服务）。你可以根据需要修改端口或地址。

> 到此所有步骤都完成了，赶紧去浏览器访问你的项目吧！
