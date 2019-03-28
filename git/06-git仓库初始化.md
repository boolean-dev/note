### git仓库初始化

#### 1. 创建新仓库create a new repository on the command line

```sh
echo "# test1" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/boolean-dev/repository.git
git push -u origin master
```

#### 2. 已存在仓库push an existing repository from the command line

```sh
git remote add origin https://github.com/boolean-dev/repository.git
git push -u origin master
```

