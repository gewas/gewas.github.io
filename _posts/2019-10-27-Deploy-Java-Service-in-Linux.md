---
layout: post
title: 学习《10天搞定Linux基础到Java项目部署》
---

老早就买了万门的 VIP，自然还是要多学学。  
一直以来个人对 Linux 那是完全不熟练，查缺补漏 ing...  
[课程链接](https://www.wanmen.org/courses/5d2eeb6ebab959d072016eec)

---

## 文件查看命令

|  命令  |                  作用                   |
| :----: | :-------------------------------------: |
|   ls   |                缩略查看                 |
| ls -a  | 查看所有文件，会显示文件名以.开头的项目 |
| ls -l  |                详细查看                 |
|   ll   |                作用同上                 |
| ls -lh |          带文件大小的详细查看           |
| ls -ld |            查看目录本身信息             |
|  tree  |             以树形结构查看              |
|  pwd   |              打印工作目录               |

注：CentOS 7 中安装 tree 命令，可以执行`yum install -y tree`

## 新建用户

- adduser username  
  新建用户

- passwd username  
  为 username 用户设置密码

## 切换用户

- su 或者 su root  
  都是切换到 root 用户，通常需要输入密码

- su username  
  切换到 username 用户，如果当前是 root 用户则无需输入密码

- su - username
  切换到 username，且将工作目录切换到用户的 home 目录

## 文件详细信息

给出示例文件信息：`-rwxr--r--. 1 root root 0 Oct 28 15:49 newshell.sh`

`-rwxr--r--.`

- 第一个字符

  - \- 文件
  - d 文件夹
  - b 块设备（硬盘设备）
  - l 链接文件(快捷方式)
  - c 串口设备（键盘、鼠标）

- 后面的 9 个字符每 3 个为一组，分别表示`r（读）w（写）x（执行）`，这 3 组分别依次为：

  - 文件所属用户（user）权限
  - 文件所属组（group）权限
  - 其他人（other）权限

- 最后一位`.`表示该文件存在`SELinux安全标签`

## 修改文件权限

- chmod

  - 参数`-R` 针对文件夹，将递归修改其子文件的权限
  - 关键字 u、g、o、a
    - 分别表示所属用户、组、其他人、所有
  - 操作符 +、-、=
  - 值 r、w、x

- 示例
  - `chmod o+x newshell.sh` 文件的 other 权限：增加可执行
  - `chmod o-w newshell.sh` 文件的 other 权限：去除写
  - `chmod o=rx newshell.sh` 文件的 other 权限：修改为读可执行

还可以使用`二进制`表示法来表示权限，表示权限的 9 位 rwxrwxrwx 字符，每一位可以由 1 或 0 代替，1 表示有效，0 表示无效。每三个数一组。如：`rwxr--r--`可理解为`111100100`，三个二进制数一组可以表示为三个十进制数`744`。  
如果我想修改权限为`rwxr-xr--`，可以执行`chmod 754 文件名`。

## 修改文件所属

- 修改所属用户：`chown [-R] 目标用户 文件或文件夹`

- 修改所属组：`chgrp [-R] 目标组 文件或文件夹`

- 同时修改：`chown [-R] 目标用户:目标组 文件或文件夹` 或 `chown [-R] 目标用户.目标组 文件或文件夹`

## Linux 目录 FHS 规范

- bin：可执行文件目录
- boot：启动文件目录
- dev：设备文件目录
- etc：配置文件目录
- root：系统管理员的 home 目录
- home：普通用户的用户目录，会为每个用户在 /home/ 中生成一个同用户名的目录作为其用户目录
- lib：库目录
- media：设备挂载点文件（U 盘）目录
- mnt：手动设备挂载点目录
- opt：大型软件安装包（源码安装包）目录
- usr：软件安装包（类似 Windows 中的 Program Files）目录。另外：usr（Unix System Resources）
- proc：进程文件目录
- sbin：系统管理员命令文件目录
- tmp：临时文件目录
- var：日志、缓存、数据文件（支持大文件）

注意：无论哪个 Linux 发行版本都**遵循 FHS 规范**，目录分类**基本一致**

## 查看系统信息

- uname：查看系统名称
- uname -i：系统配置信息
- uname -r：系统内核信息
- uname -a：系统所有信息
- lsb_release -a：系统信息
  - 如果没有 lsb_release 命令可以执行`yum install -y redhat-lsb`或`yum install -y redhat-lsb-core`安装
- cat /etc/redhat-release：系统信息
