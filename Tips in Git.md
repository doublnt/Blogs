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
git diff --staged //Same command
```

## 移除文件

首先先使用  `rm filename` 然后再使用 `git rm filename` 进行记录。

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加   .gitignore  文件，不小心把一个很大的日志文件或一堆   .a  这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用   --cached  选项

```shell
git rm --cached fileName
```

> ![](https://ws1.sinaimg.cn/large/006tNc79gy1flr6m9gerjj30ss0d276m.jpg)

## 移除在仓库中的文件夹，但不删除

比如我要移除以前以及提交到 仓库的 bundles 文件夹，但是本地并不删除它。使用

```shell
git rm -r --cached myFolder
```

## 对文件修改命名

```shell
git mv originName currentName
```

## 查看文件提交历史

### git log  常用选项

```shell
git log
git log -p //显示每次提交的内容差异
git log --stat  //显示每次提交的简略的统计信息
git log --pretty //指定使用不同于默认格式的方式提交历史。
git log --pretty=format:"%h - %an, %ar : %s"
git log --pretty=oneline / short /full/fuller // oneline 表示将每个提交放在一行内显示
git log --graph //改选项表示添加了一些ASCII 字符串来形象的展示你的分支、合并历史。
git log --grep "robert" //用来查找提交内容包含 robert 的提交
git log --abrev-commit 显示 SHA-1 的7个字符
```

![](https://ws4.sinaimg.cn/large/006tKfTcgy1flsbltblz6j312g16mk0g.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1flsbpozgh2j312k0tk7cw.jpg)

> ![](https://ws4.sinaimg.cn/large/006tNbRwgy1fmqmy8rjzuj31km09o46q.jpg)

### 限制输出长度

```shell
git log --sine=2.weeks //2可以修改
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
git checkout -- robert.md
```

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fmqn5354gcj31j208kjyd.jpg)

## 远程仓库

```shell
添加一个远程仓库
git remote add <shortname> <url>
git remote add pb https://github.com/paulboone/ticgit //添加一个远程仓库并命名为 pb
git remote -v

从远程仓库抓取与拉取
git fetch [remote-name] //git fetch origin

git push [remote-name] [branch-name]

查看远程仓库
git remote show [remote-name]
```

### 远程仓库的移除与重命名

```shell
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
git tag -a v1.2 [9fsdfdasfa]  某个提交的校验和的值
```

### 推送标签 

```shell
git push origin v1.2 推送指定标签

git push origin --tags 一次性推送把所有不在远程仓库服务器上的标签全部推送过去

在特定标签上创建一个新分支
git checkout -b [branchname] [tagname]
git checkout -b version2 v3.3.0
```

## 分支

> * 文件保存为 blob对象（保存为文件快照）
> * 一个树对象（记录目录结构和blob对象索引）
> * 一个提交对象（包含着向前述树对象的指针和所有提交信息）

查看各个分支所指的当前对象

```shell
git log --oneline --decorate //--decorate 查看各个分支当前所指的对象
```

> ![](https://ws2.sinaimg.cn/large/006tNc79gy1flwlwbwdqaj30r607kdhm.jpg)

```shell
git log --oneline --decorate --graph --all //输出你的提交历史，各个分支的指向，以及项目分支分叉的情况
```

```shell
git merge [branchname] 把 branchname 合并到当前分支
```

### 遇到冲突时分支的合并

> ![](https://ws2.sinaimg.cn/large/006tNc79gy1flwn3ti404j30vs0eeada.jpg)

任何因包含合并冲突而有待解决的文件，都会以未合并状态标识出来。 Git 会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。 出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：

```bash
<<<<<<< HEAD:index.html <div id="footer">contact : email.support@github.com</div> ======= <div id="footer">  please contact us at support@github.com </div> >>>>>>> iss53:index.html 
```

> ![](https://ws3.sinaimg.cn/large/006tNc79gy1flwn4dm26jj30qm076758.jpg)

这表示   HEAD  所指示的版本（也就是你的   master  分支所在的位置，因为你在运行 merge 命令的时候已经检出到了这个分支）在这个区段的上半部分（  =======  的上半部分），而  所要合并分支所指示的版本在   =======  的下半部分。 为了解决冲突，你必须选择使用由  =======  分割的两部分中的一个，或者你也可以自行合并这些内容。 例如，你可以通过把这段内容换成下面的样子来解决冲突：

> ![](https://ws2.sinaimg.cn/large/006tNc79gy1flwn7zy99dj30v404g3z4.jpg)

> `git push origin serverfix`
>
> ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fmqo4pbwezj31j40aathx.jpg)
>
> 或者可以将本地的分支 serverfix 推送到远程的 remotemaster 分支上
>
> `git push origin serverfix:remotemaster`

### 跟踪分支

```shell
git checkout --track origin/serverfix //将当前分支设置跟踪分支，用来 fetch 时的拉取数据
```

### 查看每一个分支的最后一次提交

```shell
git branch -v 

git branch --merged / --no-merged  查看已经合并或者未合并的分支


git branch -d / -D(强制删除)
```

### Origin 并无特殊含义

远程仓库名字   origin 与分支名字  master 一样，在 Git 中并没有任何特别的含义一样。 同时   master 是当你运行 git init 时默认的起始分支名字，原因仅仅是它的广泛使用， origin  是当你运行   git clone  时默认的远程仓库名字。 如果你运行   git clone -o booyah ，那么你默认的远程分支名字将会是   booyah/master 。

### 分支重命名

```shell
git branch -m old_branch new_branch
```

### 删除远程分支

```shell
git push origin --delete [branchName]
```



## 会丢弃暂存的情况

在当前分支（branch1）上，删除了文件的话，如果没有加入暂存，如果你使用 git checkout branch 跳到另一个分支，当你在跳回来时发现这个删除并没有起作用。

## 用户名邮箱配置

最开始使用Git时，需要配置用户名及其邮箱

```shell
git config --global user.name "Robert"
git config --global user.email robert@cnblogs.com
```

## 变基

主要是 merge 和 rebase的一些操作

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fmqotdizs4j31fm0oc7ay.jpg)

### Merge

merge 的基本原理：会把`两个分支的最新快照（C3 和 C4）`和以及`两者最近的共同祖先(C2)`进行`三方合并`，合并的结果是生成一个新的快照，并提交。

### Rebase

提取在C4 中引入的补丁和修改，然后在C3 的基础上再应用一次。可以使用 `rebase` 命令将提交到某一分支上的所有修改都移至另一分支。

对于上面的例子：

```shell
git checkout experiment
git rebase master

然后再
git checkout master
git merge experiment
```

## 分布式 Git

### 提交准则

```shell
git diff --check //找到所有的空白错误，并把它列出来。
```

## Git中的一些工具

### 查看分支引用

```shell
git show commitId/branchName // 比如 git show master

git rev-parse branchName // git rev-parse master 查看master 分支对应的SHA-1 的值
```

> ![](https://ws2.sinaimg.cn/large/006tNbRwgy1fmqpv4qqolj30vw0dmgo1.jpg)

### 查看祖先引用

`^` 符号 使用`HEAD^` 查看HEAD 的父提交

`HEAD^n`表示 HEAD的第n个提交



`~` 符号，`HEAD~` 与 `HEAD^` 是等价的，区别在于后面加数字。

`HEAD~2` 表示第一父提交的第一父提交。与 `HEAD^^` 等价

### 提交区间

`..`双点符号，可以让Git选出在一个分支而不在另一个分支上的提交。

```shell
git log master..experiment //在 experiment 分支中而不在master 分支上的。

git log master.. //如果留空 git 会自动使用HEAD 来替代
```

`^` 和 `--not`

下面的命令和上面的等价

```shell
git log ^master experiment

git log experiment --not master
```

也可以用在两个分支以上，比如

```shell
git log refA refB ^refC //查看在refA,refB 中，但是不在refC中的提交
```

`…` 三点符号，选出被两个引用中的一个包含但又不被两者同时包含的提交。

可以添加 `--left-right` 来表示在哪个分支上，

> ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fmqqk6awzcj30v60ketc8.jpg)

### 暂存 Git Stash

默认情况下，`git stash ` 只会储存已经在索引中的文件。如果指定了  `--include-untracked` 或者`-u` 标记，则会把未创建跟踪的数据也储存起来。		

```shell
git stash --include-untracked
```

为当前储存创建一个分支

```shell
git stash branch branchName
```

### 搜索

`git grep` 命令，从提交历史或工作目录中查找一个字符串或正则表达式，可以添加 `-n` 来查找输出匹配的行号。

```shell
git grep -n test
```

`--count` ,来输出哪些文件包含匹配以及每个文件包含了多少个匹配。

`-p` 查看匹配的行属于哪一个方法或者函数

`--break` ,`--heading` ,使输出更容易阅读。

### Git 日志搜索

比如想要查看 TEST常量是什么时候引入的，可以使用`-S`

```shell
git log -STEST --oneline
```

#### 行日志搜索

假如想查看zlib.c 文件中 git_test 函数的每一次变更，可以执行`git log -L :git_test:zlib.c`

```shell
git log -L :git_test:zlib.c
```

### 修改多个提交历史

通过 `git rebase -i ` 进行交互式变基

```shell
git rebase -i HEAD~3 // 尝试修改最后3次提交，实际上制定了以前的 4次。
```

使用 交互式变基，这些提交的顺序是相反的。

### filter-branch

#### 从每一个提交移除一个文件

```shell
git fliter-branch --tree-filter 'rm -f password.txt' HEAD //从整个提交历史移除一个 password.txt 文件
```

`--tree-filter` 选项在检出项目的每一个提交后运行指定的命令然后重新提交结果。

```shell
git fliter-branch --tree-filter 'rm -f *~' HEAD // 移除所有偶然提交的编辑器备份文件
```

`--all` 选项，让filter-branch 在所有分支上运行。

#### 全局修改邮箱

> ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fmqrmsnfz3j310k0o27go.jpg)

### 重置 Reset、Checkout

#### 三棵树

`HEAD`  ：上一次提交的快照，下一次提交的父节点

`INDEX` : 预期的下一次提交的快照

`Worrking Directory` : 沙盒

#### 理解 git reset

`reset`做的第一件事就是移动 HEAD 的指向。（而checkout 所做的是改变HEAD 自身）,

【移动HEAD】如果后面添加了`--soft`。那么它做的就仅仅是移动了HEAD的指向。本质上是撤消了上一次`git commit` 命令。

【更新索引】如果后面添加了`--mixed` ，则`reset` 所做的事情就到此为止了，这也是`git reset` 的默认行为。它会撤销提交，并且撤销暂存。

【更新工作目录】如果使用了`--hard` 选项，它将会继续这步。这个操作会强制覆盖你工作目录中的文件，如果该文件未提交，则会导致文件无法恢复。

`git reset file.txt` 的真正含义，当你使用

`git add file.txt` 时下面的建议命令。其实这个`reset` 命令是`git reset --mixed HEAD file.txt` 的简写形式，

* 移动HEAD 
* 让索引看起来像HEAD。

### 撤销合并

两种方式来修复

**修复引用**

如果这个不想要的合并提交只存在于你的本地仓库中，最简单且最好的解决方案是移动分支到你想要它指向的地方。大多数情况下，如果你在错误的 `gir merge` 后运行`git reset --head HEAD~` 这个重置分支指向。这个方法的缺点是它会重写历史，在一个共享的仓库中这会造成问题。*简单的说，如果其他人已经有你将要重写的提交，你应该避免使用`reset`*。

**还原提交(Revert)**

Git 给你一个生成一个新提交的选项，提交将会撤销一个已存在提交的所有修改。Git中成为**还原**。

```shell
git revert -m 1 HEAD
```

`-m 1` 标记指出 `mainline` 需要被保留下来的父结点，这时会创建一个新提交 比如`^m` 与之前的master 有同样的内容。

> ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fmqt7ejhghj31080gwq9e.jpg)

如果你在 master 分支上，使用 `git merge topic`  发现Already up-to-date.

更糟糕的是如果你在 `topic`分支上 继续提交，增加内容的话。当你要合并 topic 分支时，你发现，只合并了被还原合并之后的修改，如下图所示：

> ![](https://ws1.sinaimg.cn/large/006tNbRwgy1fmqtcirvgvj31060ccdio.jpg)

解决这个方式最好的方式，还是插销还原原始的合并，

```shell
git revert ^M

git merge topic
```

操作后的图如下图所示：

> ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fmqtec8nzbj30zm0f4jwh.jpg)

### Rerere

`git rerere` 允许你让Git记住解决一个块冲突的方法，这样在下一次看到相同冲突时，Git可以自动为你解决它。P309

 















