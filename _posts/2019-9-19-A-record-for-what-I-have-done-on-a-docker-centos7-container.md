---
layout: post
title: 我在 Docker 的 Centos 7 容器中都做了啥？
---

单纯的操作记录，目标是做一个启动即用的公司项目镜像。  
自然，敏感内容会去掉。

---

- `docker run --privileged --name=centos7 -itd centos:7 /usr/sbin/init`
- `docker exec -it centos7 bash`

---

进入容器：

- yum upgrade -y
- yum install epel-release -y
- yum install nginx -y
- systemctl start nginx
- curl 127.0.0.1
- systemctl enable nginx
- reboot

退出容器

---

- `docker commit centos7 centos7_nginx`
- `docker rm -f centos7`
- `docker run --privileged --name=centos7 -itd -p 20080:80 -p 20443:443 centos7_nginx /usr/sbin/init`
- `docker exec -it centos7 bash`

---

进入容器：

-

---
