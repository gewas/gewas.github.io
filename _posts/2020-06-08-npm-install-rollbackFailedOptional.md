---
layout: post
title: npm install 失败
---

由于总所周知的原因，npm 在国内使用十分难受。
通过修改 npm 的 registry 来修复问题。

---

## 使用管理员权限的 cmd

由于 npm 安装操作需要高级权限，所以推荐“用管理员身份运行 cmd”

## 修改 npm registry

- 直接修改全局 registry

```
npm config set registry https://registry.npm.taobao.org
```

- 临时使用指定 registry 安装

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
