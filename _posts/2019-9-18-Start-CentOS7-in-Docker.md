---
layout: post
title: 在Docker中启动CentOS 7 的注意事项
---

在 Windows 系统使用 Docker，在之前的[用 Docker 快乐的搭建工作环境](./2019-9-12-Post2-Prepare-Dev-Environment-with-Docker.md)中有说过一小部分。本篇主要是针对 CentOS 7 的使用。

---

- 首先下载 CentOS 7 的官方镜像：`docker pull centos:7`

- 然后就可以运行它了，先执行命令：`docker run --privileged -p 20022:22 -p 20080:80 -p 20443:443 -p 28080:8080 --name=centos7 -itd centos:7 /usr/sbin/init`

  - 我们使用`--privileged`配合`/usr/sbin/init`是为了防止在使用`systemctl`命令时报`Failed to get D-Bus connection: Operation not permitted`错误（这个错误十分常见，如果你只用 docker run -it centos:7 命令来启动的话。这也是我以前的使用困惑，因为并不是很懂 Linux）。

  - 然后就是用`-p`参数配置了一堆端口映射，例：`-p 20080:80`的意思就是增加本地 20080 端口到容器 80 端口的映射。

  - 最后那里的意义是设置容器的 entrypoint 到`/usr/sbin/init`，这样就可以启动 dbus 等服务，配合`--privileged`使用。

- 好了，CentOS 7 启动完毕，我们装一个 nginx 试试

  - `yum update -y`，先升级一下，`-y`参数是只如果执行过程中如果有问我 yes or no 的话，自动用 yes 应答。
  - `yum install epel-release -y`，安装 nginx 的 步骤我在[之前的文章](./2019-9-17-Setup-nginx-on-new-installed-CentOS7.md)中也有整理，这里的参数`-y`同上。
  - `yum install nginx -y`
  - `systemctl start nginx`，就是为了这个操作，我们运行 centos 容器时使用`--privileged`配合`/usr/sbin/init`。无报错运行后，nginx 正常服务。
  - 在你的电脑浏览器访问`http://127.0.0.1:20080`看看吧。

**Fantastic Docker！**
