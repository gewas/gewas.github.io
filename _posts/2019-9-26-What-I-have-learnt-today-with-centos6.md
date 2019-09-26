---
layout: post
title: 今天在 CentOS 6 上部署时学到的...
---

每天巩固就知识，每天都有新收获！

---

#### 查看已安装的 java

`rpm -qa | grep java`

#### 批量卸载安装的老 java 包

`rpm -qa | grep java | xargs rpm -e --nodeps`

#### 添加用户与为用户设定密码

`adduser 用户名`

`passwd 用户名`

#### 修改 hosts

`vi /etc/hosts`

#### 查看服务启动配置（以防火墙为例）

`chkconfig iptables --list`

#### 禁用防火墙

`service iptables stop`

`chkconfig iptables off`

`chkconfig --level 35 iptables off`

#### 生成 ssh 密钥对

`ssh-keygen`

#### ssh 添加信任

`ssh-copy-id -i <公钥路径> <用户名>@<主机>`

例：`ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.100.200.20`

#### tar 解压至指定文件夹中

`tar -zxvf ./jdk-8u211-linux-x64.tar.gz -C /usr/lib/java/`

#### 下载 sh 脚本并执行

`curl http://<域名或ip>[:端口]/path/to/shell.sh | sh`

#### 使用 scp 向别的主机复制文件

`scp -r /usr/lib/java/jdk1.8.0_211/ root@slave1:/usr/lib/java/`

说明：命令中能识别`slave1`为一台主机，是因为在`/etc/hosts`中添加了对应记录。

#### 配置 jdk 环境变量

`vi /etc/profile`

增加：

`export JAVA_HOME=/usr/lib/java/jdk1.8.0_211`
`export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar`
`export PATH=$PATH:$JAVA_HOME/bin`

保存退出后执行：

`source /etc/profile`

#### 打印当前动作目录

`pwd`

#### 清空当前终端

`clear`