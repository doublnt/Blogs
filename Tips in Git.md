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