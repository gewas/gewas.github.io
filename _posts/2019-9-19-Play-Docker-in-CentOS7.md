---
layout: post
title: 在 CentOS 7 中使用 Docker
---

哎，干这行早晚是要接触 Linux 的。工作上接到了任务，要去在 CentOS 上部署平台和应用。时间紧迫，我选 Docker。  
所以就一边实践一边整理，在 CentOS 7 上使用 Docker 的注意事项。

---

## 安装 Docker

安装 Docker 之前，我们先执行一下`yum upgrade -y`，升级一下系统的包。

然后执行命令 `curl -fsSL https://get.docker.com/ | sh`，自动化执行安装 Docker 的脚本，等待结束就行了。

脚本执行完毕，我们可以执行：

- 启动 Docker：`systemctl start docker`
- 查看状态：`systemctl status docker`
- 设置开机自启：`systemctl enable docker`

好了，目前为止就已经正常的安装 Docker 完毕了。

## 配置镜像加速器

可以通过修改 daemon 配置文件/etc/docker/daemon.json 来使用加速器

```cmd
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://阿里分配的key.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

愉快的使用 Docker 吧！
