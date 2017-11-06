# Git 中的一些问题

## 把本地新项目推送至 Github

在把本地新项目推送至`GitHub`仓库时的大致流程和步骤，首先现在`GitHub`上面新建一个项目，复制该项目的 带`.git` 后缀的地址，比如

`git@github.com:XXX/XXX.git`

然后在本地项目上 `git init` 初始化一个仓库，然后 使用 
`git add . `
`git commit -m "commit message"`
`git remote add origin git@github.com:XXX/XXX.git`
然后 `git push -u  origin master `

这时候可能会遇到
```shell
(python36) [robert@Robert-MacBook-Pro robert-learn-python (master)]$ git push -u origin master
To https://github.com/doublnt/robert-learn-python
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/doublnt/robert-learn-python'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

这时候你需要 `git pull origin master`

但是这时你又会遇到 
```shell
(python36) [robert@Robert-MacBook-Pro robert-learn-python (master)]$ git pull
fatal: refusing to merge unrelated histories
```
这时你只需要 

`git pull origin master --allow-unrelated-histories`

然后就可以直接 push 了

`git push origin master`

## 如何跳转到指定的 commit 

&emsp; 问题描述： 比如我觉得当前分支可能不适合线上的，那么我需要来一个更加安全的分支？

实现： 

```bash
git checkout -b new-branch commitId
```

对 指定的CommitId 创建新分支。