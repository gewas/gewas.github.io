---
layout: post
title: 今天在 CentOS 6 上部署时学到的...
---

学习 未知变已知

巩固 已知得新知

---

### 查看已安装的 java

`rpm -qa | grep java`

### 批量卸载安装的老 java 包

`rpm -qa | grep java | xargs rpm -e --nodeps`

### 添加用户与为用户设定密码

`adduser 用户名`

`passwd 用户名`

### 修改 hostname

`vi /etc/sysconfig/network`

### 修改 hosts

`vi /etc/hosts`

### 查看服务启动配置（以防火墙为例）

`chkconfig iptables --list`

### 禁用防火墙

`service iptables stop`

`chkconfig iptables off`

`chkconfig --level 35 iptables off`

### 生成 ssh 密钥对

`ssh-keygen`

### ssh 添加信任

`ssh-copy-id -i <公钥路径> <用户名>@<主机>`

例：`ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.100.200.20`

### tar 解压至指定文件夹中

`tar -zxvf ./jdk-8u211-linux-x64.tar.gz -C /usr/lib/java/`

### unzip 解压至指定目录中

`unzip xxx.zip -d /myDir/`

### 下载 sh 脚本并执行

`curl http://<域名或ip>[:端口]/path/to/shell.sh | sh`

### 使用 scp 向别的主机复制文件

`scp -r /usr/lib/java/jdk1.8.0_211/ root@slave1:/usr/lib/java/`

说明：命令中能识别`slave1`为一台主机，是因为在`/etc/hosts`中添加了对应记录。

### 配置 jdk 环境变量

`vi /etc/profile`

增加：

`export JAVA_HOME=/usr/lib/java/jdk1.8.0_211`
`export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar`
`export PATH=$PATH:$JAVA_HOME/bin`

保存退出后执行：

`source /etc/profile`

### 打印当前动作目录

`pwd`

### 清空当前终端

`clear`

### 查看本机 ip

`ip addr`

### 时间

查看默认时间：`date`

查看 UTC 时间：`date -u` 或 `date --utc`

查看时间并以 rfc-2822 标准输出：`date -R`

硬件时间：`hwclock --show`

硬件时间同步至系统时间：`hwclock --systohc`

### 时区

查看当前时区：`more /etc/sysconfig/clock`

修改时区：

> cp /etc/localtime /etc/localtime.bak
> cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

### yum 报错 Error: no such table: packages

使用 yum 安装时，嫌速度慢`ctrl + c`中断了以后，再次执行安装就报这个错误。

我尝试执行：`yum makecache`，结果真的就解决了。

### NTP 服务

目标：让多台主机以 master-slaves 结构保持时间同步。

操作：

1. 所有节点安装 ntp  
   `yum install -y ntp`

2. 所有节点执行

   ```text
   ntpdate 0.centos.pool.ntp.org
   hwclock --systohc
   service ntpd start
   chkconfig --add ntpd
   chkconfig ntpd on
   ```

3. master 上配置`vim /etc/ntp.conf`

   ```text
   # For more information about this file, see the man pages
   # ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

   driftfile /var/lib/ntp/drift

   # Permit time synchronization with our time source, but do not
   # permit the source to query or modify the service on this system.
   restrict default kod nomodify notrap nopeer noquery
   restrict -6 default kod nomodify notrap nopeer noquery

   # Permit all access over the loopback interface.  This could
   # be tightened as well, but to do so would effect some of
   # the administrative functions.
   restrict 127.0.0.1
   restrict -6 ::1

   # Hosts on local network are less restricted.
   restrict 10.100.200.0 mask 255.255.255.0 nomodify notrap

   # Use public servers from the pool.ntp.org project.
   # Please consider joining the pool (http://www.pool.ntp.org/join.html).
   # 中国这边最活跃的时间服务器 : http://www.pool.ntp.org/zone/cn
   server 0.cn.pool.ntp.org
   server 0.asia.pool.ntp.org
   server 3.asia.pool.ntp.org
   server 0.centos.pool.ntp.org iburst
   server 1.centos.pool.ntp.org iburst
   server 2.centos.pool.ntp.org iburst
   server 3.centos.pool.ntp.org iburst

   # allow update time by the upper server
   # 允许上层时间服务器主动修改本机时间
   restrict 0.cn.pool.ntp.org nomodify notrap noquery
   restrict 0.asia.pool.ntp.org nomodify notrap noquery
   restrict 3.asia.pool.ntp.org nomodify notrap noquery

   # Undisciplined Local Clock. This is a fake driver intended for backup
   # and when no outside source of synchronized time is available.
   # 外部时间服务器不可用时，以本地时间作为时间服务
   server  127.127.1.0
   Fudge   127.127.1.0 stratum 10

   #broadcastclient                        # broadcast client
   #broadcast 224.0.1.1 autokey            # multicast server
   #multicastclient 224.0.1.1              # multicast client
   #manycastserver 239.255.254.254         # manycast server
   #manycastclient 239.255.254.254 autokey # manycast client

   # Enable public key cryptography.
   #crypto

   includefile /etc/ntp/crypto/pw

   # Key file containing the keys and key identifiers used when operating
   # with symmetric key cryptography.
   keys /etc/ntp/keys

   # Specify the key identifiers which are trusted.
   #trustedkey 4 8 42

   # Specify the key identifier to use with the ntpdc utility.
   #requestkey 8

   # Specify the key identifier to use with the ntpq utility.
   #controlkey 8

   # Enable writing of statistics records.
   #statistics clockstats cryptostats loopstats peerstats
   ```

4. slave 上配置`vim /etc/ntp.conf`

   ```text
   # For more information about this file, see the man pages
   # ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

   driftfile /var/lib/ntp/drift

   # Permit time synchronization with our time source, but do not
   # permit the source to query or modify the service on this       system.
   restrict default kod nomodify notrap nopeer noquery
   restrict -6 default kod nomodify notrap nopeer noquery

   # Permit all access over the loopback interface.  This could
   # be tightened as well, but to do so would effect some of
   # the administrative functions.
   restrict 127.0.0.1
   restrict -6 ::1
   # 增加了这行
   restrict default ignore

   # Hosts on local network are less restricted.
   # 允许这个网段同步
   restrict 10.100.200.0 mask 255.255.255.0 nomodify notrap

   # Use public servers from the pool.ntp.org project.
   # Please consider joining the pool (http://www.pool.ntp.org/      join.html).
   #注释掉了默认配置的服务器
   #server 0.centos.pool.ntp.org iburst
   #server 1.centos.pool.ntp.org iburst
   #server 2.centos.pool.ntp.org iburst
   #server 3.centos.pool.ntp.org iburst

   #broadcast 192.168.1.255 autokey        # broadcast server
   #broadcastclient                        # broadcast client
   #broadcast 224.0.1.1 autokey            # multicast server
   #multicastclient 224.0.1.1              # multicast client
   #manycastserver 239.255.254.254         # manycast server
   #manycastclient 239.255.254.254 autokey # manycast client

   # Enable public key cryptography.
   #crypto

   includefile /etc/ntp/crypto/pw

   # Key file containing the keys and key identifiers used when       operating
   # with symmetric key cryptography.
   keys /etc/ntp/keys

   # Specify the key identifiers which are trusted.
   #trustedkey 4 8 42

   # Specify the key identifier to use with the ntpdc utility.
   #requestkey 8

   # Specify the key identifier to use with the ntpq utility.
   #controlkey 8

   # Enable writing of statistics records.
   #statistics clockstats cryptostats loopstats peerstats

   # 只允许与master同步
   server 10.100.200.20
   Fudge 10.100.200.20 stratum 8
   ```

5. 重启所有节点的 ntp 服务  
   `service ntpd restart`

6. 验证 ntp 服务  
   `watch ntpq -p`

### 查看已安装的 mysql mariaDB

`rpm -qa | grep mysql`

`rpm -qa | grep mariadb`

也可以使用 yum：

`yum list installed | grep <软件关键字>`

### 卸载已安装的 mysql mariaDB

`rpm -e --nodeps <软件名>`

`yum -y remove mysql-*`

说明：`*`是通配符

### rpm 安装 mysql

必须安装以下组件：common, libs, libs-compat, client, server

```text

// 防止出现依赖包冲突
yum install -y libaio
yum install -y perl
yum install -y numactl

// 安装mysql包
rpm -ivh ./temp/mysql-community-common-5.7.17-1.el6.x86_64.rpm
rpm -ivh ./temp/mysql-community-libs-5.7.17-1.el6.x86_64.rpm
rpm -ivh ./temp/mysql-community-libs-compat-5.7.17-1.el6.x86_64.rpm
rpm -ivh ./temp/mysql-community-client-5.7.17-1.el6.x86_64.rpm
rpm -ivh ./temp/mysql-community-server-5.7.17-1.el6.x86_64.rpm

// 为各个节点添加connector的jar包依赖
scp bigdataSoftware/mysql/mysql-connector-java.jar root@master:/usr/share/java/mysql-connector-java.jar
```

### 配置 mysql

详细的配置说明可以上网查找：

`vi /etc/my.cnf`

启动 mysql 服务：

`service mysqld start`

查看 mysql 生成的密码：

`cat mysql/logs/mysqld.log |grep password`

使用 mysql client 接入：

`mysql -uroot -p`

密码就是刚刚查出来的那个，进入后修改密码，退出，测试：

```text
mysql> SET PASSWORD = PASSWORD('mypasswd123');
mysql> exit
mysql -u root -p c
// 这一句是为了让任何主机都能用root配合密码使用mysql
mysql> grant all on *.* to 'root'@'%' identified by 'mypasswd123';
mysql> flush privileges;
mysql> exit
```

mysql 开机自启动：

`chkconfig mysqld on`

### 查看服务启动配置

查看所有的：

`chkconfig --list`

查看指定的：

`chkconfig --list | grep mysql`

### 安装当前目录中的所有.rpm

`yum -y install *.rpm`

### 循环执行指令

将本机的 /etc/xxx 文件，拷贝到 slave1 ~ slave6 主机的相同位置：

`for i in {1..6};do scp -r /etc/xxx slave$i:/etc/xxx;done`

### 快速去到 JAVA_HOME 目录

感受一下环境变量：

`cd $JAVA_HOME`
