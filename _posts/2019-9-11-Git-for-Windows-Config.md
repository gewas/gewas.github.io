---
layout: post
title: Git-for-windows 的一些配置
---

新买的办公用小米游戏本居然一直 Netwtw06 错误导致 wlan adapter 疯狂 reset，难用无比。心一狠，今天就把系统给重置了。  
重置一时爽，重新配置工作开发环境又是一下午。不过好在 wifi 问题消失了。

---

正好整理一下 git-for-windows 的几个常用配置，除了必须的 user 信息，我还需要配置：

- 防止 bash 中文乱码问题：

  - 在 bash 的 option 中把语言设置为`zh-cn`，字符编码设置为`UTF-8`
  - `git config --global core.quotepath false`

- 防止换行符 CRLF 混乱，统一使用 LF：

  - `git config --global core.autocrlf input`
  - `git config --global core.safecrlf true`

- 让 git 记录冲突的解决，遇到相同问题自动解决，方便 rebase 的执行

  - `git config --global rerere.enabled true`
