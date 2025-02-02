---
title: 用Python与Fiddler实现图书馆座位自动预约：实战经验分享
date: 2025-01-04 13:21:05
tags: [python, fiddler]
categories: 个人项目
description: 本文详细介绍了基于若依框架构建 SpringBoot 管理系统的学习过程，涵盖了框架的基本配置、功能实现及常见问题解决，帮助开发者快速上手和掌握若依框架。
cover: /img/用Python与Fiddler实现图书馆座位自动预约：实战经验分享.png
---

> 前言：
>
> 期末周图书馆座位总是供不应求，每天早上七点开始预约，不到十分钟位置就被抢光。对于我们这些爱睡懒觉的人来说，简直是噩梦！而恰好这学期我正在学习计算机网络课程，何不趁机动手写一个自动预约程序，解决这个问题呢？于是这篇博客应运而生。
>
> 免责声明：
>
> 本博客仅供计算机网络爱好者学习交流使用，请勿用于非法用途！

目录

[一、需求分析][Link 1]

[二、开发工具准备][Link 2]

[三、功能实现][Link 3]

### 一、需求分析

> 我们的目标很明确，那就是使用自动化脚本实现图书馆座位的预约。我所在的学校由于只能通过微信公众号来进行预约，所以相较于能够直接在网站上预约的学校来说，数据抓包相对复杂一点，所以才会用到 fidder，不然直接浏览器 F12 就可以直接看数据包了。

让我们来分析一下要干些什么事情：

> 1.  使用电脑登陆微信，进入公众号模拟预约
> 2.  使用 fidder 对刚才的操作进行抓包
> 3.  对数据包经行分析，提取出对应的用户登录数据以及座位预约信息
> 4.  使用 Python 的 requests 库模拟用户向图书馆服务器发送预约请求
> 5.  编辑定时器固定在早上 7 点执行程序

### 二、开发工具准备

> 明确了需求，下面来准备开发工具。

#### 1、首先先安装好 Python

[http://www.python.org/download/][http_www.python.org_download]

安装步骤其他博客已经讲的非常详细了，这里就不多说了

#### 2、然后介绍一下 Python requests 库：

> requests 是一个非常流行且易于使用的 Python 库，用于发送 HTTP 请求并与 Web 服务器进行交互。它简化了 HTTP 请求的工作流程，提供了简单的接口来进行常见的 HTTP 操作，例如 GET、POST、PUT、DELETE 等。
>
> 这次主要是用到了 post 函数向服务器发送预约请求，将 url、headers、data 这三个数据解析好之后就通过 requests.post 发送。

我这里用的是 VScode 进行开发，只需要在终端输入如下命令，即可完成 requests 库的安装：

```python
pip3 install requests
```

然后在命令行可通过导入 import 库来测试 requests 是否安装成功：

![pic_1f98df1b.png](https://api.smain.cn/pics/pic_1f98df1b.png)

这样就算是装好了

#### 3、下面介绍这次的关键抓包工具 fidder

> Fiddler 是最常用的 Web 调试工具之一。对于开发来说，前端可以通过 Fiddler 代理来调试 JS、CSS、HTML 样式。后端可以通过 Fiddler 查看请求和相应，定位问题。对于测试来说，可以通过抓包方式修改前端请求参数和模拟后端返回，快速定位缺陷。总之，不管是开发还是测试，Fiddler 都是一款工作中不可或缺的辅助利器。

官网：[https://www.telerik.com/fiddler][https_www.telerik.com_fiddler]

安装过程就是无脑下一步就行了，装好后点击 `Fiddler.exe` 运行：

![pic_704efb9d.png](https://api.smain.cn/pics/pic_704efb9d.png)

> 看上去有点复杂，不过不要慌，我们只需要会用基本的功能就可以了

下面来配置 fidder：

> 启用 HTTPS 解密
>
> 1.  在 Fiddler 菜单中，点击 `Tools > Options`。
> 2.  选择 `HTTPS` 标签页，勾选：
>
>     - `Capture HTTPS CONNECTs`
>     - `Decrypt HTTPS traffic`
>
> 3.  点击 `Actions > Trust Root Certificate`，安装 Fiddler 的证书。

![pic_464cb2c6.png](https://api.smain.cn/pics/pic_464cb2c6.png)

![pic_25ea9037.png](https://api.smain.cn/pics/pic_25ea9037.png)

> 设置抓包范围
>
> 1.  选择 `Tools > Options > Connections`。
> 2.  确保 `Allow remote computers to connect` 已勾选。
> 3.  记下 Fiddler 的监听端口（默认是 8888）。

![pic_dfdc7fd3.png](https://api.smain.cn/pics/pic_dfdc7fd3.png)

> 配置系统代理
>
> 抓取电脑端微信的流量，需要将微信的数据流经 Fiddler。
>
> 在 Windows 系统设置代理
>
> 1. 打开设置 -> 网络和 Internet -> 代理
>
> 2. 启用 使用代理服务器，并填写代理地址
>
> 地址： 127.0.0.1
>
> 端口： 8888（与 fidder 配置一致）

![pic_26713077.png](https://api.smain.cn/pics/pic_26713077.png)

> 到这里，fidder 就已经能够对微信的数据流进行监听了！

### 三、功能实现

> 现在工具都准备好了，我们马上开干！

#### 1、执行预约流程并分析抓包数据

> （1）打开微信客户端：
>
> 检查 fidder 中数据流是否检查到微信。
>
> （2）进入预约界面：
>
> 打开图书馆预约系统，执行预约操作（选择座位、时间）然后提交表单。
>
> （3）监控 fidder：
>
> 点击左侧的对应网络请求

![pic_c60379ff.png](https://api.smain.cn/pics/pic_c60379ff.png)

> 然后点击 Inspectors 查看数据包，再点击 Row 将整个请求显示为纯文本：

![pic_b1b01776.png](https://api.smain.cn/pics/pic_b1b01776.png)

> 然后查看具体响应内容：

![pic_cccc60d1.png](https://api.smain.cn/pics/pic_cccc60d1.png)

> 以下内容需要重点关注：
>
> - 目标 URL
> - Cookies

> 其中 URL 指明了目标访问地址 ，Cookies 中包含了你的用户身份信息，以及你的选座信息。

#### 2、模拟请求

> 新建一个.py 文件，引入 requests 然后将数据复制过来

```python
import requests

# URL 地址
url = " "

# 请求头，从 Raw 中提取
headers = {
    "Host": " ",
    "Connection": " ",
    "Accept": " ",
    "X-Requested-With": " ",
    "User-Agent": " ",
    "Content-Type": " ",
    "Origin": " ",
    "Sec-Fetch-Site": " ",
    "Sec-Fetch-Mode": " ",
    "Sec-Fetch-Dest": " ",
    "Referer": " ",
    "Accept-Encoding": " ",
    "Accept-Language": " ",
    "Cookie": " "
}

# 表单数据模板
data_template = {
    "code": " ",
    "data_type": " ",
    "seatdate": " ",
    "seatno"：" ",
    "seatname":" ",
    "datetime": " "
}
```

> 发送 POST 请求

```python
response = requests.post(url, headers=headers, data=data)
```

> 然后运行程序，如果看到输出状态码为 200，就表明成功与服务器交换数据，然后根据返回信息查看是否预约成功。这里推荐使用日志记录一下，这样调试起来也更方便。

```python
import logging

# 设置日志记录
logging.basicConfig(
    filename=" ",  # 设置日志文件路径
    level=logging.INFO,  # 日志等级
    format="%(asctime)s - %(levelname)s - %(message)s",  # 日志格式
    filemode='a',  # 追加写入模式
    encoding='utf-8'  # 设置编码格式为utf-8
)

# 记录响应信息
    logging.info(f"状态码: {response.status_code}")
    logging.info(f"响应内容: {response.text}")
```

> 如果响应数据没问题，那就成功了！下一步就是设置一个定时任务让这个程序在每天早上 7 点准时执行。

![pic_57297dcf.png](https://api.smain.cn/pics/pic_57297dcf.png)

> 打开任务计划程序，创建定时任务

![pic_685cedde.png](https://api.smain.cn/pics/pic_685cedde.png)

![pic_8824802d.png](https://api.smain.cn/pics/pic_8824802d.png)

![pic_e8353aa9.png](https://api.smain.cn/pics/pic_e8353aa9.png)

![pic_cf127d9c.png](https://api.smain.cn/pics/pic_cf127d9c.png)

![pic_054c03d2.png](https://api.smain.cn/pics/pic_054c03d2.png)

> 创建好后直接执行测试一下，然后取日志里看是否正常，如果一切正常，那就大功告成了！

![pic_2ae276db.png](https://api.smain.cn/pics/pic_2ae276db.png)

> 最后再次声明一下，如果没有预约成功，请你在调试的时候一定要检查清楚代码的逻辑后再运行！不要短时间内多次访问服务器！否则有可能被检测到然后封禁！

[Link 1]: #%E4%B8%80%E3%80%81%E9%9C%80%E6%B1%82%E5%88%86%E6%9E%90
[Link 2]: #%E4%BA%8C%E3%80%81%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7%E5%87%86%E5%A4%87
[Link 3]: #%E4%B8%89%E3%80%81%E5%8A%9F%E8%83%BD%E5%AE%9E%E7%8E%B0
[http_www.python.org_download]: http://www.python.org/download/
[https_www.telerik.com_fiddler]: https://www.telerik.com/fiddler
