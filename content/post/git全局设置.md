---
title: "Git 笔记"
date: 2022-06-04T16:25:38+08:00
tags: ["Git", "版本控制", "开发工具"]
categories: ["技术笔记"]
author: "谦兰君"
draft: false
description: "本文详细介绍了 Git 的全局设置方法，并整理了常用的 Git 命令，包括基础操作、分支管理、远程仓库操作等。"
weight: 100
---

### 常用 Git 命令

#### 1. 基础操作
```sh
# 初始化仓库
git init

# 克隆仓库
git clone <repository-url>

# 查看状态
git status

# 添加文件到暂存区
git add <file>
git add .  # 添加所有文件

# 提交更改
git commit -m "commit message"

# 查看提交历史
git log
```

#### 2. 分支操作
```sh
# 查看所有分支
git branch -a

# 创建新分支
git branch <branch-name>

# 切换分支
git checkout <branch-name>

# 创建并切换到新分支
git checkout -b <branch-name>

# 合并分支
git merge <branch-name>

# 删除分支
git branch -d <branch-name>
```

#### 3. 远程仓库操作
```sh
# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin <repository-url>

# 推送到远程仓库
git push origin <branch-name>

# 拉取远程仓库
git pull origin <branch-name>

# 获取远程仓库更新（不自动合并）
git fetch origin
```

#### 4. 撤销操作
```sh
# 撤销工作区文件修改
git checkout -- <file>

# 撤销暂存区文件修改
git reset HEAD <file>

# 撤销最近一次提交
git reset --soft HEAD~1  # 保留工作区和暂存区
git reset --hard HEAD~1   # 完全撤销，不可恢复
```

#### 5. 标签操作
```sh
# 查看标签
git tag

# 创建标签
git tag <tag-name>

# 创建带注释的标签
git tag -a <tag-name> -m "tag message"

# 推送标签到远程仓库
git push origin <tag-name>
git push origin --tags  # 推送所有标签
```

#### 6. 配置操作
```sh
# 查看当前配置
git config --list

# 设置用户信息（当前项目）
git config user.name "username"
git config user.email "user@email"
```

### 全局设置

```sh
# 查看全局设置
git config --global -l

# 设置全局用户名/邮箱
git config --global user.name "username"
git config --global user.email "user@email"

# 设置全局代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890

# 取消全局代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```
