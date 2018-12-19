# git版本回退

git放弃本地的修改，与远程仓库同步

```sh
#指令是下载远程仓库最新内容，不做合并 
git fetch --all
#把HEAD指向master最新版本
git reset --hard origin/master
#远程拉取更新
git pull
```

