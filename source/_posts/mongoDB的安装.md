---
title: mongoDB的安装
date: 2019-12-22 16:40:08
tags: mongoDB
---

这篇文章主要介绍 win10 安装 mongoDB4.0 过程及安装过程中一些问题的解决方法。

<!-- more -->

## 下载地址

mongoDB 的下载地址点击[这里][1]。我下载的是 msi 安装包。

## 安装过程

安装过程基本是傻瓜式的，详细的内容查看[这篇文章][2]。但是安装过程中需要注意几个问题：

- 到下图这一步时，不要勾选`Install MongoDB Compass`，因为这一步会用很长时间。`MongoDB Compass`是个图形工具，方便直接管理 mongoDB 数据。
  ![no compass][pic1]
- 安装过程中如果出现`service MongoDB failed to start，verify that you have sufficient privileges to start system services.`提示，如图：
  ![start warning][pic2]
  这种情况下使用`net start MongoDB`命令启动，会报失败。
  解决办法是：点击`Ignore`，先忽略。然后在 mongoDB 安装目录下，找到 bin 文件夹下的`mongod.cfg`文件，打开，删掉最后一行的`mp:`，重启服务，就可以成功启动。参考[这篇文章][2]

## 配置环境变量

启动服务时，有可能会提示命令 mongod 命令不存在，所以需要配置环境变量

1. 右击“我的电脑” > 打开“高级系统设置” > 打开“环境变量” > 点击用户变量中的“Path”，打开编辑环境变量的弹框 > 点击“新建”，把 mongoDB 安装目录下 bin 文件夹的完整路径复制进去。点击确定。

## 启动 mongoDB

启动 mongoDB 分为启动 mongoDB 服务（存储数据的地方）和 mongoDB 客户端（操作数据的地方）

### 启动 mongoDB 服务

> 方法一

1. 在 mongoDB 安装目录下的 bin 文件中打开命令窗口
2. 输入命令：`mongod --dbpath D:\tool\MongoDB\data`，然后启动服务。后面的路径根据你自己的 mongoDB 安装路径进行修改。
3. 如果不想在命令窗口打印日志，可先在 log 文件夹下新建名为`mongo.log`的文件，然后执行`mongod --dbpath D:\tool\MongoDB\data --logpath D:\tool\MongoDB\log\mongo.log`命令。

    注：--dbpath 是指定数据库存放目录，要注意 dbpath 前有两个“-”。如果想换访问的端口号，可以在命令后加`--port 10086`，这样就切换到了 10086 端口。

4. 命令行最后打印出：`I NETWORK [threadl] waiting for connections on port 27017`，则表示启动成功。
5. 在浏览器中输入`http://localhost:27017`,可以看到浏览器显示`It looks like you are trying to access MongoDB over HTTP on the native driver port.`

> 方法二

1. 打开命令窗口

2. 输入命令`net start MongoDB`

注意: 这种方法可能会出现一些问题

- 无法创建服务。在输入命令后提示“服务名无效”或者在任务管理器中没有找到该服务

  解决办法：以管理员身份运行 cmd，重新启动服务。

- 启动服务报“发生系统错误 5。拒绝访问。”

  解决办法：以管理员身份运行 cmd，重新启动服务。

### 启动 mongoDB 客户端

1. 打开 cmd 命令窗口
2. 执行`mongo`命令，就可以进入 mongo 的客户端，进而进行数据库操作。

### 一些问题

1. mongod命令默认启动的地址是localhost，也就是本地地址（也可以用127.0.0.1）链接。如果想要切换成本机IP，可以按照下面的方式:

        /* 启动服务，bind_ip参数跟的是要切换的IP */
        mongod --dbpath D:\tool\MongoDB\data --logpath D:\tool\MongoDB\log\mongo.log --bind_ip 10.0.1.109

        /* 启动客户端，命令后面直接跟IP:端口，如果是默认27017端口，则不用加 */
        mongo 10.0.1.109

[1]: https://www.mongodb.org/dl/win32
[2]: https://blog.csdn.net/qq_20084101/article/details/82261195
[pic1]: https://img-blog.csdn.net/2018091918200759?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R0YTA1MDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70
[pic2]: https://img-blog.csdnimg.cn/20190818135529950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDE5ODk2NQ==,size_16,color_FFFFFF,t_70
