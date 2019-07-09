# Centos安装docker

## 1. 安装器准备

### 1.1 移除旧版本

```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

### 1.2 安装必要系统工具

```sh
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 1.3 添加软件信息源

```sh
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 1.4 更新yum缓存

```sh
sudo yum makecache fast
```

## 2. 安装docker

### 2.1 安装 Docker-ce

```sh
sudo yum -y install docker-ce
```

### 2.2 设置为开机自启

```sh
systemctl enable docker
```

### 2.2 启动

```sh
sudo systemctl start docker
```

### 2.3 测试

```sh
docker run hello-world
```

## 3 镜像加速

### 3.1 配置镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，我使用的是网易的镜像地址：**http://hub-mirror.c.163.com**。

新版的 Docker 使用 `/etc/docker/daemon.json`来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：

```json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

### 3.2 重新加载文件

```sh
systemctl daemon-reload
```

### 3.3 重启docker服务

```sh
systemctl restart docker
```



> 文章参考：
>
> [CentOS Docker 安装](https://www.runoob.com/docker/centos-docker-install.html)
>
> [Centos中配置docker镜像加速](https://blog.csdn.net/zj7321/article/details/80197488)

