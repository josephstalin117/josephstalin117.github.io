---
layout: post
title:  "git分支命名约定"
date:   2019-12-11 20:35:35 +0800
categories: command
---

master 分支

1. master 为主分支，也是用于部署生产环境的分支，确保master分支稳定性
2. master 分支一般由develop以及hotfix分支合并，任何时间都不能直接修改代码

develop 分支

1. develop 为开发分支，始终保持最新完成以及bug修复后的代码
2. 一般开发的新功能时，feature分支都是基于develop分支下创建的

feature 分支

1. 开发新功能时，以develop为基础创建feature分支
2. 分支命名: feature/ 开头的为特性分支， 命名规则: feature/user_module、 feature/cart_module

release分支

release 为预上线分支，发布提测阶段，会release分支代码为基准提测

```
当有一组feature开发完成，首先会合并到develop分支，进入提测时，会创建release分支。
如果测试过程中若存在bug需要修复，则直接由开发者在release分支修复并提交。
当测试完成之后，合并release分支到master和develop分支，此时master为最新代码，用作上线。
```

hotfix 分支
1. 分支命名: hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似
2. 线上出现紧急问题时，需要及时修复，以master分支为基线，创建hotfix分支，修复完成后，需要合并到master分支和develop分支

日志规范
type: 本次 commit 的类型，诸如 bugfix docs style 等  
scope: 本次 commit 波及的范围  
subject: 简明扼要的阐述下本次 commit 的主旨，在原文中特意强调了几点  
```
1. 使用祈使句
2. 首字母不要大写
3. 结尾无需添加标点
```
body: 同样使用祈使句，在主体内容中我们需要把本次 commit 详细的描述一下，比如此次变更的动机，如需换行，则使用 |  
footer: 描述下与之关联的 issue 或 break change  

Type的类别说明：
```
feat: 添加新特性
fix: 修复bug
docs: 仅仅修改了文档
style: 仅仅修改了空格、格式缩进、都好等等，不改变代码逻辑
refactor: 代码重构，没有加新功能或者修复bug
perf: 增加代码进行性能测试
test: 增加测试用例
chore: 改变构建流程、或者增加依赖库、工具等
```