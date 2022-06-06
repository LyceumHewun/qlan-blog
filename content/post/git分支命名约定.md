---
title: "Git 分支命名约定"
date: 2022-06-06T14:47:16+08:00
tags: ["编码规范"]
categories: []
author: "谦兰君"
draft: false
description: ""
weight: 100
---

**Git** 分支分为集成分支、功能分支和修复分支，分别命名为 *develop*、*feature* 和 *hotfix*。

### （1）*master* 分支

*master* 为主分支，也是用于部署生产环境的分支。为了确保*master*分支稳定性，一般由*develop*、*hotfix*分支合并，任何时间都不能直接修改代码。

### （2）*develop* 分支

*develop*为开发分支，始终保持最新完成以及bug修复后的代码。一般开发新功能时，都是基于*develop*分支创建*feature*分支的。

### （3）*feature* 分支

*feature*为功能分支，分为**版本功能分支**和**非版本功能分支**。

1. **版本功能**分支是根据版本需求分出来的功能分支，这时候命名为：

***feature/{$version}/{$issue_id}_{$description}*** 

2. 非版本功能分支则是指不跟版本一起上线的功能或者一些不紧急的 bugs，这时候命名为：

***feature/{$username}/{$issue_id}_{$description}***

### （4）*hotfix* 分支

*hotfix*为修复分支，表示修复紧急线上***BUG***的分支（不紧急的***BUG***归为功能分支），命名为：

***hotfix/{$username}/{$issue_id}_{$description}***



## 变量

- **$version** 版本号
- **$username** 开发者
- **$issue_id** issues的id
- **$description** 分支功能描述亦或issues的标题



## 参考文献

<nobr>*[1] 科比08. Git分支命名规范(2017-06-05)\[2022-06-06]. https://www.cnblogs.com/kobe1991/p/6944747.html*</nobr>

<nobr>*[2] josephlin. Git分支命名约定(2019-12-11)\[2022-06-06]. http://josephlin.org/git-styleguide.html*<nobr>

