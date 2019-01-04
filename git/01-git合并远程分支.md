### git合并远程分支

```sh
# 1. 把源码clone到本地
git clone [gitsite  git远程网址]
# 2. 在本地建立一个和远程分支相同的本地分支
git checkout -b dev origin/dev
# 3. 切换到主分支master
git checkout master
# 4. 将本地的dev合并到本地主分支master中
git merge dev
# 5. 将本地主分支master同步到远程
git push origin master
# 6.删除远程分支
git push origin --delete dev
```

