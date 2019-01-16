## 一、前言
在上一篇博客中，讲述了在服务器上安装多个tomcat，现在这篇博客要讲是安装nginx
## 二、配置Nginx的安装环境
安装Nginx需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境，需要安装gcc：`yum install gcc-c++ `，安装完gcc后，才可以进行下一步的安装
## 三、编译安装

 ### 1. 解压缩

```sh
将Nginx安装包nginx-1.8.0.tar.gz拷贝至服务器上
解压缩安装包：
`tar -zxvf nginx-1.8.0.tar.gz`
`cd nginx-1.8.0`
```

###  2. 配置安装参数
		参数如下:



```sh
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi

```

注：**上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录**

###  3. 编译安装
```sh
make
make  install
```

#四、启动Nginx

　　`cd /usr/local/nginx/sbin/`
　　`./nginx `
## 五、停止Nginx
`cd /usr/local/nginx/sbin`
`./nginx -s stop`
此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。 
方式2，完整停止(建议使用)：
`cd /usr/local/nginx/sbin`
`./nginx -s quit`
此方式停止步骤是待nginx进程处理任务完毕进行停止。
## 六、重启Nginx
方式1，先停止再启动（建议使用）：
对nginx进行重启相当于先停止nginx再启动nginx，即先执行停止命令再执行启动命令。
如下：
`./nginx -s quit./nginx`

方式2，重新加载配置文件：
当nginx的配置文件nginx.conf修改后，要想让配置生效需要重启nginx，使用-s reload不用先停止nginx再启动nginx即可将配置信息在nginx中生效，如下：
`./nginx -s reload`
## 七、开机自启Nginx

 ### 1. 编写shell文件

```sh
vi /etc/init.d/nginx

#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/var/run/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1
fi
   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}
   RETVAL=$?
   echo
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   return $RETVAL
}
# Stop nginx daemons functions.
stop() {
        echo -n $"Stopping $prog: "
        killproc $nginxd
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
status)
        status $prog
        RETVAL=$?
        ;;
*)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
exit $RETVAL
```



![自启命令](https://img-blog.csdn.net/20180409152813251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb2xlYW5pbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### 2. 设置文件访问权限

```sh
`chmod a+x /etc/init.d/nginx`   (a+x ==> all user can execute  所有用户可执行)
```

### 3. 加入到自动列表中

```sh
vi /etc/rc.local
 
加入一行  /etc/init.d/nginx start   保存并退出，下次重启会生效。
```