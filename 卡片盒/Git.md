# 概述
> Git 是一个免费的、开源的分布式版本控制系统，旨在快速高效地处理从小型到非常大的项目的所有内容。

# 核心概念
- 工作区(*Working Directory*),电脑本地硬盘目录，一般是项目当前目录
- 暂存区(*stage*),存放在.git目录下的index文件，我们把暂存区有时也叫作索引(index)
- 版本库(*Repository*),工作区里的隐藏目录.git，它就是Git的版本库（也称为本地库）

# 常用指令
- git init 初始化本地库,使这个文件夹作为本地库
- git status 查看文件的状态
- git checkout -b branch_name 新建分支并切换
- git stash 将当前工作区和暂存区的内容隐藏起来(用于当前工作流被打断的情况)
- git stash pop 将隐藏的内容恢复

# 工作流程
增加新功能
1. 从远端拉取 dev 分支 `git pull dev`
2. 在 dev 分支上创建 feature 打头的新功能分支,命名规则 `feature/cart_module`
3. 在 feature 分支上开发完毕后,将特性合并到 dev 分支
4. [参见:Git 分支开发规范](https://cloud.tencent.com/developer/article/1809754)

紧急修复 Bug
1. 从 master 建立 hotfix 分支,命名规则 `hotfix/xxx`
2. 将 hotfix 分支合并到 master,并上线到生成环境
3. 把 hotfix 分支合并到 dev 分支
# Commit Message
目前使用较为广泛的提交信息规范是 [Angular Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)
```code
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```
## Header
type(必须): 用于说明 commit 的类别，只允许使用下面7个标识
- feat：新功能（feature）
- fix：修补 bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- chore：构建过程或辅助工具的变动

scope(可选): 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
subject(必须): 是 commit 目的的简短描述，不超过50个字符。

## Body
Body 部分是对本次 commit 的详细描述，可以分成多行。
有两个注意点。
1. 使用第一人称现在时，比如使用 `change` 而不是 `changed` 或 `changes`。
2. 应该说明代码变动的动机，以及与以前行为的对比。

## Footer
用于标识不兼容改动 `BREAKING CHANGE` ,关闭 Issue 或标注贡献者等
[详见](https://cubox.pro/my/card?id=ff80808184d2af120184d7d3df0d7903)