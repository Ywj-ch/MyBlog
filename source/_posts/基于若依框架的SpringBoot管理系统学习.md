---
title: 基于若依框架的SpringBoot管理系统学习
date: 2024-10-24 15:03:12
tags: [springboot]
categories: 学习笔记
description: 本文详细介绍了基于若依框架构建 SpringBoot 管理系统的学习过程，涵盖了框架的基本配置、功能实现及常见问题解决，帮助开发者快速上手和掌握若依框架。
cover: /img/基于若依框架的SpringBoot管理系统学习.png
swiper_index: 2
---

#### 前言

> 在现代企业开发中，快速搭建高效、稳定的管理系统是一个常见的需求。而若依框架作为一个基于 SpringBoot 的开源管理系统框架，凭借其模块化设计、便捷的代码生成工具以及优秀的前端整合方案，成为了许多开发者的首选。对于初学者来说，若依框架不仅是学习 SpringBoot 开发的良好切入点，也是深入理解后台管理系统开发流程的最佳实践平台。
>
> 本文将结合学习若依框架的实际过程，从框架搭建到核心功能实现，对其中的重要知识点和开发技巧进行深入解析，帮助读者快速上手并掌握若依框架的使用技巧，同时为开发自己的管理系统打下坚实的基础。

#### 一、什么是若依？

##### 1.概述

> 若依框架（RuoYi）是一个基于 SpringBoot 和 Vue 的快速开发平台，常用于构建后台管理系统。它采用前后端分离的架构，前端使用 Vue.js，后端使用 SpringBoot，数据库则支持多种类型（如 MySQL、MariaDB 等）。框架集成了一些主流的开源组件，如 MyBatis、Redis、Druid、Swagger 等，使得开发人员能够快速搭建和扩展项目功能。

> gitee 地址：
>
> 后端 [https://gitee.com/y_project/RuoYi][https_gitee.com_y_project_RuoYi]
>
> 前端 [https://gitee.com/y_project/RuoYi-Vue][https_gitee.com_y_project_RuoYi-Vue]

> 技术版本：
>
> - JDK >= 1.8
> - MySQL >= 5.7
> - Maven >= 3.0
> - Node >= 12
> - Redis >= 3

##### 2.若依框架的特点

> - 模块化设计：内置了用户管理、角色权限、菜单管理、操作日志、定时任务等常见的管理系统功能模块，可以快速搭建企业级应用。
> - 前后端分离：前端与后端独立开发和部署，提高了开发效率和用户体验。
> - 丰富的技术栈集成：集成了多种常用的技术和工具（如 MyBatis 做数据持久化，Redis 做缓存，Druid 作为数据库连接池），简化了开发过程。
> - 高扩展性：代码结构清晰，支持二次开发，可以根据业务需求进行自定义扩展。

##### 3.若依框架的目录结构

> 若依的后端项目主要分为六个模块，他们之间的依赖关系如下图所示：

![pic_59f382bd.png](https://api.smain.cn/pics/pic_59f382bd.png)

下面我们来分析一下每个模块的具体功能

![pic_e9c40141.png](https://api.smain.cn/pics/pic_e9c40141.png)

![pic_1c2520b9.png](https://api.smain.cn/pics/pic_1c2520b9.png)

![pic_1e9d8b33.png](https://api.smain.cn/pics/pic_1e9d8b33.png)

![pic_aa8ccf3b.png](https://api.smain.cn/pics/pic_aa8ccf3b.png)

> 从上面的几张图我们可以看出若依的主要框架包括 admin、common、framework 和 system，至于 quartz 和 generator 它们并不是必须的，但是有了它们，可以大大加快我们程序的开发速度，后面我们会讲到如何使用。

若依的配置文件主要放在-admin 包中

![pic_16f83402.png](https://api.smain.cn/pics/pic_16f83402.png)

接下来是项目相关的数据库表

![pic_3d67c75b.png](https://api.smain.cn/pics/pic_3d67c75b.png)

然后让我们来看看前端的项目结构

> 这张图完整了列出了前端项目的框架，但是如果你像我一样重心放在后端开发，那其实只需要了解其中一部份模块的功能就行了，下面这张图列出了后端人员经常需要用到的几个模块。

![pic_39357e97.png](https://api.smain.cn/pics/pic_39357e97.png)

下面们来分析一下若依后端的具体代码

> 首先从 Controller 开始，若依框架中的 Controller 都继承了 BaseController 类，实现了基本的数据分页展示功能、用户登录相关方法、以及请求成功或者失败后的相关处理。可以说，若依的这个 BaseController 类基本上实现了业务开发所要用到的大多数常用功能。

![pic_8f67fd18.png](https://api.smain.cn/pics/pic_8f67fd18.png)

> 然后是我们比较关心的返回对象的封装，若依将返回对象分为了两类，一类是分页查询返回对象，另一类是增删改查返回对象。

![pic_9d46515e.png](https://api.smain.cn/pics/pic_9d46515e.png)

![pic_b2b81c37.png](https://api.smain.cn/pics/pic_b2b81c37.png)

```java
@PreAuthorize("@ss.hasPermi('manage:partner:list')")
    @GetMapping("/list")
    public TableDataInfo list(Partner partner)
    {
        startPage();
        List<PartnerVo> list = partnerService.selectPartnerVOList(partner);
        return getDataTable(list);
    }
```

```java
@GetMapping("/captchaImage")
    public AjaxResult getCode(HttpServletResponse response) throws IOException
    {
        AjaxResult ajax = AjaxResult.success();
        boolean captchaEnabled = configService.selectCaptchaEnabled();
        ajax.put("captchaEnabled", captchaEnabled);
        if (!captchaEnabled)
        {
            return ajax;
        }
```

> 值得一提的是@PreAuthorize 注解是 Spring Security 的一个权限认证注解，它与前端联调之后能够实现权限控制访问。

```java
@PreAuthorize("@ss.hasPermi('manage:partner:edit')")
```

```java
<el-button link type="primary" @click="resetPassword(scope.row)" v-hasPermi="['manage:partner:edit']">重置密码</el-button>
```

![pic_c06ad374.png](https://api.smain.cn/pics/pic_c06ad374.png)

前后端交互流程：

![pic_e4586998.png](https://api.smain.cn/pics/pic_e4586998.png)

![pic_471e64aa.png](https://api.smain.cn/pics/pic_471e64aa.png)

#### 二、若依框架的项目搭建

##### 1.后端项目初始化配置

###### 配置数据库

> 将后端代码克隆的本地后会有两个 sql 脚本，先创建好你的数据库然后修改数据库连接配置

```java
spring:
  datasource:
    druid:
      url: jdbc:mysql://localhost:3306/ruoyi?
      useUnicode=true&characterEncoding=utf8&serverTimezone=UTC
      username: your_db_username
      password: your_db_password
```

> ruoyi 是数据库的名字，然后执行下面的 sql 脚本就行了。

![pic_3c915a97.png](https://api.smain.cn/pics/pic_3c915a97.png)

###### 配置 Redis 缓存

```java
spring:
  redis:
    host: localhost
    port: 6379
    password: your_redis_password
```

> 注意：启动项目之前记得一定要把 Redis 打开，不然会报错。

###### 依赖管理

> 若依框架的依赖管理主要使用 Maven，`pom.xml`文件配置了各种依赖，包括 Spring Boot、MyBatis、Redis、Swagger 等。以下是主要依赖的说明：

```java
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

##### 2.启动项目

> 启动后端项目：在`ruoyi-admin`目录下执行`RuoYiApplication`类中的`main`方法，启动 SpringBoot 后端服务。
>
> 启动前端项目：在`ruoyi-ui`目录下，执行以下命令安装依赖并启动前端服务：

```java
npm install
npm run dev
```

##### 3.界面展示

> 成功启动后页面会自动跳转到如下登录界面，登录密码默认是 admin123，若依的密码使用了 MD5 加密存储，所以在数据库里面是看不到的。

![pic_44fe88e3.png](https://api.smain.cn/pics/pic_44fe88e3.png)

##### 4.功能详讲

> 若依框架的菜单功能是后台管理系统的核心部分，涵盖了系统管理、监控、工具等多种模块。以下是若依系统默认的菜单结构和功能介绍：
>
> ###### 1. 系统管理
>
> 主要用于管理系统中的基础设置和权限控制，包括用户、角色、菜单等内容。主要功能如下：
>
> - 用户管理  
>   管理系统中的用户信息，包括新增、编辑、删除用户，以及为用户分配角色和重置密码等操作。
> - 角色管理  
>   定义系统中的角色，并为角色分配菜单权限。可以设置角色的权限范围，控制其可访问的菜单项。
> - 菜单管理  
>   用于配置系统的菜单显示和访问权限，支持添加、编辑、删除菜单项。可以设置菜单的类型（目录、菜单、按钮）、排序、图标等信息。
> - 部门管理  
>   维护系统的组织架构，支持树形结构显示部门信息。可以新增、编辑、删除部门，以及查看部门成员。
> - 岗位管理  
>   定义系统中的岗位信息，用于关联用户岗位。可以新增、编辑、删除岗位。
> - 字典管理  
>   管理系统中的字典数据，如状态、性别等常用数据项。可以为每个字典项设置标签和值。
> - 参数设置  
>   用于管理系统中的配置参数，可以动态修改参数值而无需重启服务。
> - 通知公告  
>   发布和管理系统内的公告信息，支持查看公告的详细内容。
>
> ###### 2. 系统监控
>
> 主要用于监控系统的运行状态、日志记录等内容，帮助运维人员管理和排查系统问题。主要功能包括：
>
> - 在线用户  
>   显示当前在线的用户列表，可以查看用户登录信息和强制下线。
> - 定时任务  
>   管理系统中的定时任务（调度任务），支持任务的新增、编辑、删除，以及手动执行和暂停任务。若依默认使用 Quartz 作为定时任务调度器。
> - 操作日志  
>   记录用户的操作行为，如新增、编辑、删除等操作，便于审计和问题追踪。
> - 登录日志  
>   记录用户的登录历史，包括登录时间、IP 地址、登录状态等信息。
> - 系统日志  
>   显示系统的运行日志，便于运维人员查看和分析问题。
>
> ###### 3. 系统工具
>
> 提供一些开发和运维相关的工具，帮助开发人员和运维人员提高工作效率。主要功能包括：
>
> - 代码生成  
>   根据数据库表结构自动生成基础的代码模板，包括 Controller、Service、Mapper、Entity、Vue 页面等，大幅提高开发效率。支持自定义生成策略。
> - 系统接口  
>   提供系统 API 的在线文档，基于 Swagger 生成，支持在线测试和接口文档查看。
> - 表单构建  
>   可视化创建表单，支持生成 Vue 表单页面，简化前端开发工作。

![pic_0ed0d635.png](https://api.smain.cn/pics/pic_0ed0d635.png)

##### 5.若依的代码生成器功能

> 这里就用到了若依的 generator 模块，若依框架的`generator`模块是一个代码生成器模块，用于自动生成基础的 CRUD 代码和页面模板。这个模块极大地提高了开发效率，尤其是在开发后台管理系统时，减少了重复性的工作。下面是对`generator`模块的详细讲解：

###### 1. `generator`模块的主要功能

> - 根据数据库表结构生成代码：可以通过解析数据库表结构自动生成对应的 Java 代码和 Vue 前端代码，包括 Controller、Service、Mapper、Entity（实体类）以及前端的页面（如列表页、表单页）。
> - 支持自定义模板：可以根据需求自定义代码生成的模板，灵活控制生成代码的风格和内容。
> - 可配置性强：支持根据生成选项定制生成的代码，例如是否生成分页查询、是否生成插入、删除、更新方法等。
> - 大幅减少重复开发工作量：生成的代码可以作为基础模板，开发人员只需根据业务需求做个性化调整即可。

###### 2. `generator`模块的使用步骤

> ###### 2.1 进入代码生成器界面
>
> - 在后台管理系统中，点击左侧菜单的“系统工具” -> “代码生成”即可进入代码生成器的管理界面。
> - 该界面显示了所有从数据库中读取的表信息，包括表名、表描述、创建时间等

###### ![pic_33165d4b.png](https://api.smain.cn/pics/pic_33165d4b.png)

> ###### 2.2 配置生成参数
>
> 选择需要生成代码的数据库表后，点击“生成配置”按钮进入配置页面，可以对生成的代码进行一些参数配置：
>
> - 生成模块名：可以指定代码所属的模块名称。
> - 生成包路径：用于指定生成的 Java 代码的包路径。
> - 生成模板：用于指定表的类型，分别有单表、树表和主子表。
> - 上级菜单：分配到指定菜单下。

![pic_9ae66709.png](https://api.smain.cn/pics/pic_9ae66709.png)

> ###### 2.3 生成代码
>
> - 配置完成后，点击“生成代码”按钮，系统会根据配置自动生成对应的 Java 代码和 Vue 前端代码。
> - 生成的代码会自动打包成一个压缩文件，包含了后端和前端的代码模板，下载后可以解压并将代码复制到项目中进行个性化修改。
> - 能够对生成的代码进行预览。

![pic_8d7736e5.png](https://api.smain.cn/pics/pic_8d7736e5.png)

###### 3. 生成的代码结构

> 生成的代码分为后端代码和前端代码，具体结构如下：
>
> ###### 3.1 后端代码
>
> - Controller 层：负责处理 HTTP 请求，将请求分发给 Service 层，并将响应结果返回给前端。生成的 Controller 代码包含了基本的 CRUD 操作。
> - Service 层：负责业务逻辑的处理。生成的 Service 代码通常包含接口和实现类。
> - Mapper 层：MyBatis 的 Mapper 接口，用于执行数据库操作。生成的 Mapper 包含基本的增删改查方法。
> - Entity 类：表示数据库表的实体类，与表的结构对应。
> - Mapper XML：MyBatis 的 SQL 映射文件，包含了 CRUD 操作的 SQL 语句。
>
> ###### 3.2 前端代码
>
> - 列表页面（`xxx.vue`）：用于显示数据列表，支持分页、查询、增删改等操作。
> - 新增/编辑页面：用于添加和修改数据的表单界面。
> - 前端接口文件（`api/xxx.js`）：定义了前端对后端的请求方法，如获取列表数据、添加、删除等操作。

###### 4. 代码生成器的自定义

> 若依的`generator`模块支持自定义模板和生成逻辑，满足不同项目的定制化需求：
>
> ###### 4.1 自定义代码生成模板
>
> - 若依的代码生成器使用 Freemarker 模板引擎，所有的生成模板都存放在`resources/vm`目录下。
> - 可以根据项目的规范和需求，修改默认模板或者添加新的模板来定制生成的代码格式。
>
> ###### 4.2 自定义生成策略
>
> - 可以通过修改代码生成器的实现逻辑，改变代码生成的规则。例如，可以在生成时自动添加业务逻辑或增加额外的注释信息。

###### 5. 常见的使用场景

> - 快速构建 CRUD 模块：对于后台管理系统，CRUD 模块是常见需求，代码生成器可以大幅减少重复的 CRUD 开发工作。
> - 原型开发：在项目初期快速生成系统的原型，帮助团队进行功能验证和需求讨论。
> - 代码模板统一：通过定制模板，可以使项目中的代码风格和规范统一，提高代码的可维护性。

#### 三、自定应化若以框架

> 前言：这部分内容我将会根据实际业务需求来改造若依框架，内容参考 B 站黑马若依“帝可得”项目教程，对若依原理感兴趣的可以去看一下原视频，个人感觉讲的还是比较清楚的。

###### 1.构建基本框架

> 首先让我们来创建三张表，下面是建表语句：

```java
CREATE TABLE `tb_region` (
  `id` INT AUTO_INCREMENT COMMENT '主键id' PRIMARY KEY,
  `region_name` VARCHAR(255) NOT NULL COMMENT '区域名称',
  `create_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `create_by` VARCHAR(64) COMMENT '创建人',
  `update_by` VARCHAR(64) COMMENT '修改人',
  `remark` TEXT COMMENT '备注'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='区域表';

-- 插入测试数据
INSERT INTO `tb_region` (`region_name`,`remark`) VALUES ('北京市朝阳区','北京市朝阳区'), ('北京市海淀区','北京市海淀区'), ('北京市东城区','北京市东城区');

CREATE TABLE `tb_partner` (
  `id` INT AUTO_INCREMENT COMMENT '主键id' PRIMARY KEY,
  `partner_name` VARCHAR(255) NOT NULL COMMENT '合作商名称',
  `contact_person` VARCHAR(64) COMMENT '联系人',
  `contact_phone` VARCHAR(15) COMMENT '联系电话',
  `profit_ratio` INT COMMENT '分成比例',
  `account` VARCHAR(64) COMMENT '账号',
  `password` VARCHAR(64) COMMENT '密码',
  `create_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `create_by` VARCHAR(64) COMMENT '创建人',
  `update_by` VARCHAR(64) COMMENT '修改人',
  `remark` TEXT COMMENT '备注'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='合作商表';

-- 插入测试数据
INSERT INTO `tb_partner` (`partner_name`, `contact_person`, `contact_phone`, `profit_ratio`, `account`, `password`) VALUES
('合作商A', '张三', '13800138000', 30, 'a001', 'pwdA'),
('合作商B', '李四', '13912345678', 25, 'b002', 'pwdB');

CREATE TABLE `tb_node` (
  `id` INT AUTO_INCREMENT COMMENT '主键id' PRIMARY KEY,
  `node_name` VARCHAR(255) NOT NULL COMMENT '点位名称',
  `address` VARCHAR(255) NOT NULL COMMENT '详细地址',
  `business_type` INT COMMENT '商圈类型',
  `region_id` INT COMMENT '区域ID',
  `partner_id` INT COMMENT '合作商ID',
  `create_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `create_by` VARCHAR(64) COMMENT '创建人',
  `update_by` VARCHAR(64) COMMENT '修改人',
  `remark` TEXT COMMENT '备注',
  FOREIGN KEY (`region_id`) REFERENCES `tb_region`(`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`partner_id`) REFERENCES `tb_partner`(`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='点位表';


-- 插入测试数据
-- 假设区域ID为1对应'北京市朝阳区'，合作商ID为1对应'合作商A'
INSERT INTO `tb_node` (`node_name`, `address`, `business_type`, `region_id`, `partner_id`) VALUES
('三里屯点位', '北京市朝阳区三里屯路', 1, 1, 1),
('五道口点位', '北京市海淀区五道口', 2, 2, 2);
```

> 然后是这三张表之间的关系：可以看到区域表跟点位表是一对多的关系，合作商表与点位表也是一对多的关系，一个区域里面或者是一个合作商都可以拥有多个点位。

![pic_a0178a50.png](https://api.smain.cn/pics/pic_a0178a50.png)

> 然后我们使用若依的代码生成器来生成区域管理相关代码

![pic_95dbd639.png](https://api.smain.cn/pics/pic_95dbd639.png)

![pic_19566da6.png](https://api.smain.cn/pics/pic_19566da6.png)

![pic_a1491b66.png](https://api.smain.cn/pics/pic_a1491b66.png)

> 然后将生成的代码分别导入到你的前后端项目中，main 文件夹中生成的后端代码，vue 文件夹中生成的前端代码，下面的三条 sql 脚本是若依框架的动态菜单表，将这三张表导入以后就可以使用若依的菜单管理功能对这三张菜单进行动态管理。

![pic_72575bda.png](https://api.smain.cn/pics/pic_72575bda.png)

![pic_0f44480a.png](https://api.smain.cn/pics/pic_0f44480a.png) ![pic_97c44b56.png](https://api.smain.cn/pics/pic_97c44b56.png)

> 然后启动项目进入到区域管理菜单模块，基础界面如下图所示：

![pic_61267570.png](https://api.smain.cn/pics/pic_61267570.png)

###### 2.自定义框架

###### 2.1 问题分析

> 可以看到，若依帮我们生成的代码还是有很多问题的：

> 1.首先就是列表名 id，用 id 来当列表名其实是若依默认的，但这显然是不合适的的，所以我们要把 id 改为序号。
>
> 2.其次就是我们在刚刚建表时就对这三张表分析过了，区域表显然是更点位表有关联的，但是在页面上并没有体现出来，我们是想要在区域菜单中展示对应区域有多少个点位的！
>
> 3.除了基本的修改和删除，如果我还想要增加一个按键叫做“查看详情”，要求点击该按键后弹出该区域的详细信息因该怎么实现？

###### 2.2 解决方法

> 下面我们针对这些问题一一来解决

> 首先是序号名称问题，我们将 lable 标签中的值修改掉就好了

```java
<el-table-column label="序号" align="center" prop="id" />
```

> 然后是对应区域点位数量的问题，这个问题我们需要将其拆封成两个部分，第一部分是后端的视图封装，我们定义一个 RegionVo 类，来封装所要在前端展示的数据，由前面的分析可知我们希望展示点位数据，所以先继承原有的 Regon 类，然后在此基础上添加一个 nodeCount 变量用来存储对应的点位数量（变量名的定义因该尽量跟数据库对应数据类型的名字一样，并采用驼峰式命名风格）

```java
<!-- 使用驼峰命名法转换字段 -->
<setting name="mapUnderscoreToCamelCase" value="true"/>
```

![pic_549657fb.png](https://api.smain.cn/pics/pic_549657fb.png)

> 然后就是实现点位数量的查询，这里有两种实现思路
>
> （1）同步存储：在区域表中有点位数的字段，当点位发生变化时，同步区域表中的点位数。
>
> - 优点：由于是单表查询操作，查询列表效率最高。
> - 缺点：需要在点位增删改时修改区域表中的数据，有额外的开销，数据也可能不一致。
>
> （2）关联查询：编写关联查询语句，在 mapper 层封装。
>
> - 优点：实时查询，数据 100%正确，不需要单独维护。
> - 缺点：SQL 语句较复杂，如果数据量大，性能比较低。
>
> 由于区域和点位表，记录个数都不是很多，所以我们采用关联查询这种方案。

> SQL 查询：先聚合统计每个区域的点位数，然后与区域表进行关联查询

```java
select r.id, r.region_name, r.remark, ifnull(n.node_count, 0) as node_count
from  tb_region r left join
(select region_id, count(*) as node_count
from tb_node group by region_id) n on r.id = n.region_id
```

> 先在查询中测试这段 sql 代码，运行结果没问题后再放入 mapper.xml 中。

![pic_dd2a6197.png](https://api.smain.cn/pics/pic_dd2a6197.png)

![pic_c878a3e5.png](https://api.smain.cn/pics/pic_c878a3e5.png)

> 加 where 的原因是为了动态地生成 SQL 查询的 `WHERE` 子句，它可以帮助避免因条件拼接时的语法错误，自动处理 `WHERE` 关键字和 `AND` 的连接问题。

> 然后即使基本的 mapper 层、service 层、servviceipml 层、controller 层的编写，这里就快速过一下代码。

RegionMapper

```java
/**
 * 查询区域管理列表
 * @param region
 * @return RegionVo集合
 */
public List<RegionVo> selectRegionVoList(Region region);
```

IRegionService

```java
/**
 * 查询区域管理列表，带点位数量
 *
 * @param region 区域管理
 * @return RegionVo集合
 */
 public List<RegionVo> selectRegionVoList(Region region);
```

RegionServiceImpl

```java
/**
 * 查询区域管理列表
 * @param region
 * @return RegionVo集合
 */
@Override
public List<RegionVo> selectRegionVoList(Region region) {
    return regionMapper.selectRegionVoList(region);
}
```

RegionController

```java
/**
 * 查询区域管理列表
 */
@PreAuthorize("@ss.hasPermi('manage:region:list')")
@GetMapping("/list")
public TableDataInfo list(Region region)
{
    startPage();
    List<RegionVo> voList = regionService.selectRegionVoList(region);
    return getDataTable(voList);
}
```

> 到此后端代码改造完成，需要注意的是，除了在 Controller 层中对原有的 list 方法进行修改以外，剩下的几层都是新增方法，不要把之前的方法改掉！不然由于有一些接口比如数据导出，还是使用的 selectRegionList（）方法就会报错！

> 接下来让我们把目光转向前端，由于后端已经数据封装好了，这里只需要在 el-table-column 标签中指定 prop 属性即可。

region/index.vue

```java
<el-table-column label="点位数" align="center" prop="nodeCount" />
```

> 修改完成后重新启动项目查看界面，没有问题点位数据成功显示！

![pic_884739fb.png](https://api.smain.cn/pics/pic_884739fb.png)

> 没有显示成功的可以打开 F12 然后点击一下重置按钮看一下数据返回没有，如果有下面的数据就证明后端没有问题，检查前端代码写错没有，如果连数据都没有返回，就证明是后端代码哪里写错了，以我的经验来看一般都是数据库查询的时候出问题。

> 遇到 bug 的时候不要怕！根据报错信息一步步寻找错误，学会使用断点进行调试和 log.info(需用引入 SLF4J 依赖)输出日志信息，修改 bug 的过程就是你对项目结构进一步加深了解的过程！

![pic_c82ce8dc.png](https://api.smain.cn/pics/pic_c82ce8dc.png)

> 最后我们来填第三个坑，实现查看详情按钮。

> 还是先来分析一下，我们最后想要实现下图的效果，点击查询详情后弹出下面的提示框，提示框中显示了区域名称和包含的点位，显然点位是用一个 List<NodeVo> 对象来存储的，这里又涉及到一个坑了，设备数量在哪里呢？这里为了不再增加复杂度并且考虑到连贯性的关系，就不再对点位表进行改造了，我们修改一下需求，只显示区域包括的点位名称，不显示设备数量。

![pic_9bd96557.png](https://api.smain.cn/pics/pic_9bd96557.png)

> 要查询点位数据，我们就要用到前端的 node.js 中的 listNode 方法

```java
// 查询点位管理列表
export function listNode(query) {
  return request({
    url: '/manage/node/list',
    method: 'get',
    params: query
  })
}
```

> 该方法调用后端的 list 方法查询点位数据

```java
/**
 * 查询点位管理列表
 */
 @PreAuthorize("@ss.hasPermi('manage:node:list')")
 @GetMapping("/list")
 public TableDataInfo list(Node node)
    {
        startPage();
        List<Node> list = nodeService.selectNodeList(node);
        return getDataTable(list);
    }
```

> 然后我们在`<script>`中来编写 `getRegionInfo` 方法，并绑定按键

```html
<el-button
  link
  type="primary"
  @click="getRegionInfo(scope.row)"
  v-hasPermi="['manage:node:list']"
  >查看详情</el-button
>
```

> 记得引入 js 代码

```javascript
import {
  listRegion,
  getRegion,
  delRegion,
  addRegion,
  updateRegion,
} from "@/api/manage/region";
import { listNode } from "@/api/manage/node";
import { loadAllParams } from "@/api/page";
```

```javascript
/* 查看详情按钮操作 */
const nodeList = ref([]);
const regionInfoOpen = ref(false);
function getRegionInfo(row) {
  // 查询区域信息
  reset();
  const _id = row.id;
  getRegion(_id).then((response) => {
    form.value = response.data;
  });
  // 查询点位列表
  loadAllParams.regionId = row.id;
  listNode(loadAllParams).then((response) => {
    nodeList.value = response.rows;
  });
  regionInfoOpen.value = true;
}
```

> 下面是对这一段 javascript 代码的详细解释，本问题的关键所在就是这段代码，有了这段代码才能够拿到数据。
>
> 1. `const nodeList = ref([]);`
>
> - 这行代码定义了一个响应式变量 `nodeList`，初始值为空数组。
> - `ref` 是 Vue.js Composition API 提供的用于创建响应式数据的函数，`nodeList` 会随着值的变化自动触发视图更新。
>
> 2. `const regionInfoOpen = ref(false);`
>
> - 定义了另一个响应式变量 `regionInfoOpen`，初始值为 `false`。
> - 这个变量用来控制区域信息详情的显示状态，当 `regionInfoOpen` 为 `true` 时，详情会显示。
>
> 3. `function getRegionInfo(row) { ... }`
>
> - 定义了一个函数 `getRegionInfo`，用于处理查看区域详情的操作。
> - 参数 `row` 是一个对象，通常表示区域的某一行数据。
>
> 4. `reset();`
>
> - 调用了一个 `reset()` 函数，可能用于重置表单或页面状态，以清除之前的数据。具体的实现不在这段代码中。
>
> 5. `const _id = row.id`
>
> - 获取 `row` 对象的 `id` 属性值，并将其存储在 `_id` 变量中，用于后续的查询操作。
>
> 6. `getRegion(_id).then(response => { ... })`
>
> - 调用 `getRegion` 方法获取区域的详细信息，`_id` 作为查询参数。
> - `getRegion` 应该是一个返回 Promise 的函数，使用 `.then` 方法处理异步响应。
> - 如果请求成功，`response.data` 会赋值给 `form.value`，其中 `form` 应该是另一个 `ref` 响应式变量，用于绑定表单数据。
>
> 7. `loadAllParams.regionId = row.id;`
>
> - 更新 `loadAllParams` 对象的 `regionId` 属性为当前区域的 `id`，用于查询点位列表。
>
> 8. `listNode(loadAllParams).then(response => { ... })`
>
> - 调用 `listNode` 方法获取与当前区域相关的点位列表，`loadAllParams` 作为查询参数。
> - `listNode` 也是一个返回 Promise 的函数，异步处理响应结果。
> - 请求成功时，将返回的数据（`response.rows`）赋值给 `nodeList.value`，更新点位列表。
>
> 9. `regionInfoOpen.value = true;`
>
> - 将 `regionInfoOpen` 的值设置为 `true`，表示区域信息详情的弹窗或面板应该显示。

> 理解完了这段代码后，最后就是编写一个提示框，绑定对应的数据即可！

```javascript
<!-- 查看详情对话框 -->
    <el-dialog title="区域详情" v-model="regionInfoOpen" width="500px" append-to-body>
      <el-form-item label="区域名称" prop="regionName">
          <el-input v-model="form.regionName" disabled />
      </el-form-item>
      <label>包含点位：</label>
      <el-table :data="nodeList">
          <el-table-column label="序号" type="index" width="50" align="center" />
          <el-table-column label="点位名称" align="center" prop="nodeName" />
      </el-table>
    </el-dialog>
```

> 欧克，做完这一步之后就可以重新启动项目进行测试了。

![pic_92b16645.png](https://api.smain.cn/pics/pic_92b16645.png)

> 可以看到点击查看详情后成功返回提示框！

> 到此，自定义若依框架的一些基本步骤都已经演示完毕，如果你认真的看到了这里并且自己动手试验了，那么我相信你一定对若依框架有了更深刻的了解和认识，当然，由于本文只是若依的基础入门讲解，所以只是定义了一些基础的功能，若依能做的远远不止这些，例如定时任务调度，数据可视化报表等等，笔者也是水平有限，如果有什么错误敬请指正，希望能和大家一起学习一起进步！

#### 四、结语

> 到这里，若依框架的基本使用方法已经介绍完毕。需要注意的是，虽然若依的代码生成器极大地提高了开发效率，方便快捷地生成基本的增删改查功能，但对于较为复杂的业务场景，例如多表联查或复杂的业务逻辑处理，仍需要开发者自行进行调整和优化。这也为我们提供了更多的灵活性，可以根据实际需求对代码进行更深入的定制化开发。

> 前路漫漫道阻且长，砥砺前行与君共勉！

[https_gitee.com_y_project_RuoYi]: https://gitee.com/y_project/RuoYi
[https_gitee.com_y_project_RuoYi-Vue]: https://gitee.com/y_project/RuoYi-Vue
