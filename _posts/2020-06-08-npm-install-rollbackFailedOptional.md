---
layout: post
title: npm install 失败
---

由于总所周知的原因，npm在国内使用十分难受。
通过修改npm的registry来修复问题。

---

## 修改npm registry

```
npm config set registry http://registry.npm.taobao.org/
```
