---
layout: post
title: 学习《10天搞定Linux基础到Java项目部署》
---

老早就买了万门的 VIP，自然还是要多学学。  
一直以来个人对 Linux 那是完全不熟练，查缺补漏 ing...  
[课程链接](https://www.wanmen.org/courses/5d2eeb6ebab959d072016eec)

---

## 文件查看命令

- `ls` 缩略查看
- `ls -a` 查看所有文件，会显示文件名以.开头的项目
- `ls -l` 详细查看
- `ll` 作用同上
- `ls -lh` 带文件大小的详细查看
- `ls -ld` 查看目录本身信息
- `tree` 以树形结构查看
  - 安装 tree 命令，可以执行`yum install -y tree`

## 新建用户

- adduser username  
  新建用户

- passwd username  
  为 username 用户设置密码

- userdel -r username  
  完全删除 username 用户

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

## 目录与路径的操作

- pwd：打印当前工作目录
- cd：改变工作目录
- mkdir：创建目录
  - 参数-p，可实现一次创建多级目录
    - `mkdir -p a/b/c`
    - `mkdir -p d/{d1,d2,d3}`，创建 d 目录，且在 d 目录中创建 d1、d2、d3 三个目录
- rmdir：删除空目录
  - 智能删除空目录
  - 使用-p 参数，可以一次删除多级空目录  
    `rmdir -p a/b/c`
- rm -rf：强制删除文件（夹）
- touch：创建一个空白文件或更新已存在文件的最后访问时间
- file：查看文件的类型
- cat：现实文本文件的内容
  - more
  - less

## 绝对路径与相对类路径

- 绝对路径：一定由`/`写起，从根出发
- 相对路径：路径从当前路径出发
  - . 代表此层目录
  - .. 代表上一层目录
  - \- 代表前一个工作目录
  - ~ 代表当前用户的 home 目录
  - ~username 代表 username 用户的 home 目录

## PATH

PATH 环境变量，在 Linux 中 PATH 的分隔符是`:`

- 查看当前 PATH
  - echo \$PATH
- 修改当前 PATH，增加一个目录`/usr/java`
  - PATH=\$PATH":/usr/java"

## 查看文件内容

- cat：由第一行开始显示文件内容
- tac：从最后一行开始显示，tac 与 cat 正好相反
- nl：显示的时候，输出行号
- more：分页显示内容
- less：同 more，但是还可以用方向键向前翻页
- head：只看头几行
- tail：只看尾几行
  - tail -f：保持跟踪打印文件的最后几行，该操作通常用于看日志
- od：以二进制方式读取文件内容
- strings：打印文件中能打印的字符内容

## umask

当我们通过 touch 或 mkdir 产生新文件（夹）时，会默认配置权限：

- 文件：666
- 文件夹：777

但是默认下实际操作情况为：

- 文件：644
- 文件夹：755

就是因为 umask 会过滤一道，可以使用`umask`或`umask -S`查看当前设置的 umask 值。

`umask 000`，可以设置为 umask 不过滤，但是通常不推荐修改。

## 隐藏权限

`chattr [+-] <attr> 文件或目录`

- `chattr +a d1`，给 d1 目录添加 append 权限，使 d1 目录中只能添加不能删除，多用于日志
- `chattr +i f2`，设定 f2 文件不能被删除、改名、设定链接关系，同时不能写入或新增内容

## 查看文件类型

使用`file 文件`命令来查看文件类型。

## 高级创建用户

`useradd [-g <主组>] [-G <附加组1,附加组2...>] [-r] <用户名>`

-g 指定用户所属主组  
-G 指定用户所属附加组  
-r 等效于--system ，创建一个系统用户

注：默认不会给系统用户创建 home 目录

## 用户相关命令

- id 显示当前用户信息
- passwd 修改当前用户的密码
  - 普通用户修改密码会受到密码强度检测限制
  - root 可以给其他用户修改密码，可无视密码强度限制
- whoami 查看自己
- groups 查看所属组

## 用户信息

查看用户信息：`cat /etc/passwd`

用户密码信息：`cat /etc/shadow`

## passwd 高级使用

改命令还可以：

- `passwd -l <username>` 锁定用户
- `passwd -u <username>` 解锁用户
- `passwd -d <username>` 使该用户能够无密码使用

## sudo

直接使用 root 用户做所有操作是很不安全的，通常都会新建专有用户去做不同的工作。但是经常又需要高权限去做一些操作，不可能总`su`来`su`去。**sudo**命令就是来临时使用 root 权限去执行命令。

必要操作：将用户添加到`/etc/sudoers`文件中，才能使用户能够使用 sudo

## 组操作

- 添加组：`groupadd <groupName>`

  - /etc/group 中存放着组信息

- 删除组：`groupdel <groupName>`

## 修改用户组

`usermod [-g <主组>] [-G <附加组1,附加组2...>] <用户名>`

## 搜索指令位置

搜索 ls 在哪里：`which ls`

搜索 uname 在哪里：`which uname`

## 查找文件

命令：`find <查找位置> <查找参数>`

示例：

- `find . -name "s*"` 在当前目录模糊查找名字以 s 开头的文件
- `find / -iname "java*" -user root` 在根目录查找，大小写不敏感且名字以 java 开头且所有者是 root 的文件

参数：

- -name 文件名
- -iname 文件名大小写不敏感
- -user 所属用户
- -group 所属组
- -or 或条件

## 查找内容 grep

grep, Global Regular Expression Print

命令：`grep [options] pattern file`

参数：

- -n 显示行号
- -i 忽略大小写
- -w 单词完整匹配

示例：

- 从文件中查找关键字：`grep 'linux' target.txt`
- 找出以 u 开头的行：`grep ^u test.txt`
- 找出以 hat 结尾的行：`grep hat$ article.txt`

注：如果正则表达式中含有空格，一定要用引号将正则表达式部分包起来
