# git 常用语句

### 1. 创建仓库

#### git init

```sh
#使用当前目录初始化为git仓库
git init
#使用指定目录作为git仓库
git init newrepo
```

#### git clone

```sh
#从远程仓库将仓库拷贝到当前目录
#格式
git clone <repo>
#示例
git clone git://github.com/schacon/grit.git

#从远程仓库拷贝项目到指定目录
#格式
git clone <repo> <directory>
#示例
git clone git://github.com/schacon/grit.git mygit
```

#### git add

```sh
#将README添加到本地缓存中
git add README
```

#### git status

```sh
#查看当前仓库的状态，是否文件被修改或者添加
git status
```

#### git diff

```sh
#查看文件被改动的位置
git diff
```

#### git commit

```sh
#将缓存添加到本地仓库中
git commit -m 'frist commit'
```

#### git rm

```sh
#从仓库移除该目录
git rm flie
```

#### git mv

```sh
#移动、重命名文件
git mv README README.md
```

#### git branch

```sh
#列出当前本地分支
git branch
#查看所有分支（远程和本地）
git branch -a
#查看远程分支
git branch -r
#创建一个新分支newBranch
git branch newBranch
#删除分支newBranch
git branch -d newBranch
```

#### git checkout

```sh
#切换分支master
git checkout master
```

#### git merge

```sh
#合并分支newBranch到主分支
git merge newBranch
```

#### git log

```sh
#查看提交历史
git log
#查看历史记录简洁版本
git log --oneline
#查看历史记录带分支合并
git log --oneline --graph
#逆向显示日志
git log --reverse --oneline
#查看指定用户提交的代码
git log --author=Linus --oneline -5
#查看指定日期的历史
git log --oneline --before={3.weeks.ago} --after={2010-04-18} --no-merges
```

#### git push

```sh
#将本地dev分支提交到远程仓库dev分支
git push origin dev:dev
```

#### get pull

```sh
#更新远程代码到本地
git pull
```



#### git config

```SH
#查看用户名
git config user.name
#查看邮箱
git config user.email
#修改用户名和邮箱地址命令
git config user.name "name"
git config user.email "email"
#修改全局用户名和邮箱地址
git config --global user.name "name"
git config --global user.email "email"
```

