---
layout: post
title:  "git分支命名约定"
date:   2019-12-11 20:35:35 +0800
categories: command
---

## 概述
约定式提交规范是一种基于提交信息的轻量级约定。 它提供了一组简单规则来创建清晰的提交历史； 这更有利于编写自动化工具。 通过在提交信息中描述功能、修复和破坏性变更， 使这种惯例与`SemVer`相互对应。
提交说明的结构如下所示：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]

<类型>[可选 范围]: <描述>

[可选 正文]

[可选 脚注]
```

日志规范
`type`: 本次`commit`的类型，诸如`fix`, `docs`, `style`等  
`scope`: 本次`commit`波及的范围
`subject`: 简明扼要的阐述下本次`commit`的主旨，在原文中特意强调了几点

1. 使用祈使句
2. 首字母不要大写
3. 结尾无需添加标点


`body`: 同样使用祈使句，在主体内容中我们需要把本次`commit`详细的描述一下，比如此次变更的动机，如需换行，则使用`|`  
`footer`: 描述下与之关联的`issue`或`break change`  

`Type`的类别说明：

- `feat`: 添加新特性，类型为`feat`的提交表示在代码库中新增了一个功能（这和语义化版本中的`MINOR`相对应）。
- `fix`: 修复bug，表示在代码库中修复了一个 bug（这和语义化版本中的 PATCH 相对应）
- `docs`: 仅仅修改了文档，例如修改`README`文件、`API`文档等；
- `style`: 仅仅修改了空格、格式缩进等等，不改变代码逻辑
- `refactor`: 代码重构，没有加新功能或者修复bug，例如修改代码结构、变量名、函数名等但不修改功能逻辑；
- `perf`: 增加代码进行性能测试
- `test`: 增加测试用例
- `chore`: 改变构建流程、或者增加依赖库、工具等，用于对非业务性代码进行修改，例如修改构建流程或者工具配置等；
- `build`: 用于修改项目构建系统，例如修改依赖库、外部接口或者升级 Node 版本等
- `BREAKING CHANGE`: 在脚注中包含`BREAKING CHANGE`: 或`<type>(scope)`后面有一个`!`的提交，表示引入了破坏性`API`变更（这和语义化版本中的`MAJOR`相对应）。 破坏性变更可以是任意`<type>`提交的一部分。`

其它提交类型在约定式提交规范中并没有强制限制，并且在语义化版本中没有隐式影响（除非它们包含`BREAKING CHANGE`）。 可以为提交类型添加一个围在圆括号内的范围，以为其提供额外的上下文信息。例如`feat(parser): adds ability to parse arrays.`。

## 一些例子

包含了描述并且脚注中有破坏性变更的提交说明
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

包含了`!`字符以提醒注意破坏性变更的提交说明
```
feat!: send an email to the customer when a product is shipped
```

包含了范围和破坏性变更`!`的提交说明
```
feat(api)!: send an email to the customer when a product is shipped
```

包含了`!`和`BREAKING CHANGE`脚注的提交说明
```
chore!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

不包含正文的提交说明
```
docs: correct spelling of CHANGELOG
```

包含多行正文和多行脚注的提交说明
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```


## 分支命名

`master`分支

1. `master`为主分支，也是用于部署生产环境的分支，确保`master`分支稳定性
2. `master`分支一般由`develop`以及`hotfix`分支合并，任何时间都不能直接修改代码

`develop`分支

1. `develop`为开发分支，始终保持最新完成以及`bug`修复后的代码
2. 一般开发的新功能时，`feature`分支都是基于`develop`分支下创建的

`feature`分支

1. 开发新功能时，以`develop`为基础创建`feature`分支
2. 分支命名: `feature/`开头的为特性分支， 命名规则: `feature/user_module`、 `feature/cart_module`

`release`分支

`release`为预上线分支，发布提测阶段，会`release`分支代码为基准提测。当有一组`feature`开发完成，首先会合并到`develop`分支，进入提测时，会创建`release`分支。如果测试过程中若存在`bug`需要修复，则直接由开发者在`release`分支修复并提交。当测试完成之后，合并`release`分支到`master`和`develop`分支，此时`master`为最新代码，用作上线。


`hotfix`分支
1. 分支命名: `hotfix/`开头的为修复分支，它的命名规则与`feature`分支类似
2. 线上出现紧急问题时，需要及时修复，以`master`分支为基线，创建`hotfix`分支，修复完成后，需要合并到`master`分支和`develop`分支


