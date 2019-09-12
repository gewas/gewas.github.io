---
layout: post
title: 用Docker快乐的搭建工作环境
---

你还在痛苦的安装 MySql5.6 5.7 8.0，多个项目依赖的数据库版本不一致怎么办？要装一堆软件难以管理，卸载还有残留怎么办？  
尝试 Docker 吧！

---

我目前的工作环境就需要我安装 Mysql 5.7 版本，以及 Redis，外加一个自己玩的 RabbitMQ。懒得装这些软件，以及配套的语言环境。Windows 使用 Docker 只需以下几步即可完成环境的搭建：

1. 去[Docker 官网](https://www.docker.com)注册账号，安装 Docker Desktop。
1. 下载各个镜像：
   1. docker pull mysql:5.7
   1. docker pull redis
   1. docker pull rabbitmq:management
1. 逐个启动：

   > docker run --name mysql57 --restart=always -p 3306:3306 -p 33060:33060 -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
   >
   > docker run --name redis --restart=always -p 6379:6379 -d redis redis-server --appendonly yes
   >
   > docker run --name rabbitmq --restart=always --hostname my-rabbit -p 4369:4369 -p 5671:5671 -p 5672:5672 -p 15671:15671 -p 15672:15672 -p 25672:25672 -e RABBITMQ_DEFAULT_USER=username -e RABBITMQ_DEFAULT_PASS=password -d rabbitmq:management

1. 至此，mysql 已经在 3306 端口，redis 在 6379 端口，rabbit 在 5672 以及 15672 端口待命了。

[Docker Hub](https://hub.docker.com)还有很多镜像等着你去探索，自己也可以制作镜像分享给他人使用。
