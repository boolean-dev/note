## git拉取远程指定分支到本地

以前写自己模块的时候，曾经建了一个本地分支dev，然后这个分支很久没有使用了，最近又要使用这个分支dev，然后要将远程仓库master分支上的内容同步到我本地dev分支

```sh
# 切换分支到本地分支
git checkout dev
# 拉取更新远程主仓库数据
git pull origin master
# 本地仓库关联远程仓库dev
# git branch --set-upstream-to [远程分支] [本地分支]
git branch --set-upstream-to origin/dev dev
```



