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
#### git 完全取消最近的提交并回退到上一个状态
```
git reset --hard HEAD~1
```
#### 刷新DNS缓存
```
cmd中运行： ipconfig/flushdns
```
