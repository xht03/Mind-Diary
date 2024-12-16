---
title: Git简明教程
date: 2024-02-03 17:14:27
tags:
- original
categories:
- The Missing Semester
---

## 常用 Git 指令

**基础设置：**

- 查看所有配置：`git config -l`

- 查看系统配置：`git config --system --list` 或 `git config --system -l`

- 查看用户配置：`git config --global --list`

  ---

- 设置用户名和邮箱：`git config --global user.name "你的用户名"` 

  ​								`git config --global user.email "你的邮箱地址"`

- 删除用户名和邮箱：`git config --global --unset user.name`

  ​								`git config --global --unset user.email`

  ---

**常规操作：**

- 查看所有文件状态：`git status`

- 查看指定文件状态：`git status [filename]`

- 查看日志：`git log`

- 查看引用记录：`git relog`

  ---

- 初始化（在当前目录下新建一个代码库）：`git init`

- 添加文件到暂存区：`git add [文件名]`

- 添加工作区所有文件到暂存区：`git add .`

- 提交更改：`git commit` 或 `git commit -m "消息内容"`

- 推送到远程仓库：`git push` 或 `git push [远程仓库名称] [本地分支名称]:[远程分支名称]`

- 克隆仓库：`git clone [URL]`

- 拉取远程变更：`git pull` 或 `git pull [远程仓库名称] [远程分支名称]`

- 获取远程变更：`git fetch [远程仓库名称]`

  

  > *`git fetch`会获取最新的提交，但不会自动合并到当前的工作分支*

  ---

**分支管理：**

- 查看所有分支：`git branch`

- 创建新的分支：`git branch [new branchname]`

- 重命名分支：`git branch -m [old branchname] [new branchname]`

- 删除分支：`git branch -d [branchbname]`

- 切换分支：`git checkout [branchname]` 或 `git switch [branchname]`

- 合并分支：`git merge [source branchname]`

  

  > *请在目标分支执行`git merge`，使**目标分支**同步**源分支**的更改*

  ---

- 撤销修改：`git checkout .`

  > 回到最后一次提交的状态，撤销后续的修改。
  
- 回退到某次提交：`git checkout [commit-hash]`

  > *你不再在任何特定的分支上，而是直接在一个特定的提交上。*
  >
  > *此时，你可以自由地查看这个提交的代码，进行实验性的修改，甚至创建新的提交。这些修改和提交不会影响任何分支，除非你创建一个新的分branch并将这些提交添加到那里。*

- 重置到某次提交：`git reset --hard [commit-hash]`

  > *reset会用永久地丢弃后续的提交历史。请确保不再需要之后的提交。*

  ---

**远程仓库：**

- 添加远程仓库：`git remote add [远程仓库名称] [远程仓库URL]`
- 查看远程仓库：`git remote -v`
- 删除远程仓库：`git remote rm [远程仓库名称]`



## Git 原理

Git 是**分布式版本控制**。不同于**本地版本控制**和**集中式版本控制**（如SVN），每个人都拥有全部的代码。所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在连网时push到相应的服务器或其他用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用。这样不会因为服务器损坏或者网络问题，造成不能工作的情况。（详见：[狂神聊Git](https://mp.weixin.qq.com/s/Bf7uVhGiu47uOELjmC5uXQ)）



Git 本地有三个工作区域，外加远程的git仓库，总共分为四个工作区域：

- **工作目录**  (Working Directory)
- **暂存区** (Stage/Index)
- **资源库** (Repository或Git Directory)
- **远程的 git 仓库** (Remote Directory)

![relationship](https://ref.xht03.online/202412101913504.png)



