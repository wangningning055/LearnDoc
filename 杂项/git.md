---
title: git
categories: git
---

#### git连接不上，刷新dns
```shell
本地：
git config --global --unset http.proxy
git config --global --unset https.proxy
挂梯子：
git config --global http.proxy 127.0.0.1:7890
git config --global https.proxy 127.0.0.1:7890
//7890是梯子的端口号
```
