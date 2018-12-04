# git提交代码

```sh
#新建分支
git branch yanjiantao
#查看所有分支
git branch -a
#切换分支
git checkout yanjiantao
#查看当前文件状态
git status
#更新远程代码到本地
git pull
#添加文件到缓存区
git add file
#提交文件到本地仓库
git commit -a -m "massage"
#提交本地yanjiaotao分支到远程仓库yanjiantao分支
git push origin yanjiantao:yanjiantao
#放弃本地修改
git checkout -- filepathname
#合并某分支到当前分支
git merger yanjiantao
#删除分支
git branch -d yanjiantao
```

