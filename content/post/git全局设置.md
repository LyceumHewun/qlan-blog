---
title: "Git全局设置"
date: 2022-06-04T16:25:38+08:00
tags: []
categories: []
author: "谦兰君"
draft: false
description: ""
weight: 100
---

### 查看全局设置

```sh
git config --global -l
```

### 设置全局用户名/邮箱

```sh
git config --global user.name "username"
git config --global user.email "user@email"
```

### 设置全局代理

```sh
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

