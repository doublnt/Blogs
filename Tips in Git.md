# Git 中的一些问题

## 前言

&emsp; 这篇文章主要是介绍我在使用Git中的有一些忘记了，但是很重要的命令。

## 如何跳转到指定的 commit

&emsp; 问题描述： 比如我觉得当前分支可能不适合线上的，那么我需要来一个更加安全的分支？

实现： 

```bash
git checkout -b new-branch commitId
```

对 指定的 `CommitId `创建新分支。

## 克隆远程仓库的时候自定义本地仓库名字

```shell
git clone http://github.com/xx myName
```

## **使用 git status -s 可以看见状态的简写版本，如下所示**

> ![](https://ws3.sinaimg.cn/large/006tNc79gy1flr5pg02s0j30oy09sabj.jpg)

## 查看已暂存的文件差异比较

以前一直不知道，当暂存之后使用 git diff 就无效了，今天发现竟然还有这个

(在未 Commit 的状态下！)

```shell
git diff --cached
git diff --staged
```

## 移除文件

首先先使用  `rm filename` 然后再使用 `git rm filename` 进行记录。

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加   .gitignore  文件，不小心把一个很大的日志文件或一堆   .a  这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用   --cached  选项

```shell
git rm --cached deleteName
```

> ![](https://ws1.sinaimg.cn/large/006tNc79gy1flr6m9gerjj30ss0d276m.jpg)

## 查看文件提交历史

```shell
git log
git log -p //显示每次提交的内容差异
git log --stat  //显示每次提交的简略的统计信息
git log --pretty=format:"%h - %an, %ar : %s"
```

![](https://ws4.sinaimg.cn/large/006tKfTcgy1flsbltblz6j312g16mk0g.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1flsbpozgh2j312k0tk7cw.jpg)

限制输出长度

```shell
git log --sine=2.weeks
git log -<n> 显示 前 n 条提交
git log --author 指定作者的提交
git log -SFunctionName 可以列出那些添加或移除了某些字符串的提交
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1flsc0e2m8hj312g0mgq8j.jpg)

## 撤销相关操作

撤销刚刚的一个提交

```shell
git commit --amend //由于commit 信息写错，好像经常用
```

## 取消暂存文件

当你使用 git add * 时，发现添加了不应该暂存的文件，可以用下面的来取消暂存

```shell
git reset HEAD <file>

git checkout -- <file> //取消对一个未加入暂存区文件的修改
```

## 远程仓库

```shell
添加一个远程仓库
git remote add <shortname> <url>
git remote add pb https://github.com/paulboone/ticgit

从远程仓库抓取与拉取

git fetch [remote-name] //git fetch origin


git push [remote-name] [branch-name]


查看远程仓库
git remote show [remote-name]

远程仓库的移除与重命名
git remote rename <originName> <afterName>

git remote rm <remote-name>
```

## 标签

```shell
git tag 列出标签
```

Git 使用两种主要类型的标签： 轻量标签(lightweight) 与 附注标签(annotated)。

一个轻量标签很像一个不会改变的分支-它只是一个特定提交的引用。

附注标签是存储在Git数据库中的一个完整对象。它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用GNU Privacy Guard （GPG） 签名与验证。

### 附注标签 

```shell
git tag -a v1.4 -m 'my version 1.4'

git show 可以看到标签信息对应的提交信息
```

### 轻量标签

```shell
git tag v1.4-lw
```

### 给以前的提交打标签

```shell
git tag -a v1.2 9fsdfdasfa  某个提交的校验和的值
```

### 推送标签

```shell
git push origin v1.2 推送指定标签

git push origin --tags 一次性推送把所有不在远程仓库服务器上的标签全部推送过去

在特定标签上创建一个新分支
git checkout -b [branchname] [tagname]
git checkout -b version2 v3.3.0
```

