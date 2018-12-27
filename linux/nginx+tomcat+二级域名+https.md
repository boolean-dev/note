### 1.添加域名解析

在腾讯云或者阿里云中添加域名解析，解析到具体得ip

![1545915339038](E:\workspace\笔记\note\image\2018-12-27-01.png)



### 2. 购买SSL证书

在腾讯云或者阿里云中购买相应的SSL，基本上每个SSL只能解析一个HTTPS，所以如果你有多个二级域名设置HTTPS的话，需要申请多个SSL

![1545915611111](E:\workspace\笔记\note\image\2018-12-27-02.png)

### 3.安装多个tomcat

#### 3.1下载tomcat安装包放入服务器中

从网上下载tomcat的linux安装包，然后利用WinSCp软件拖入到服务器，我放得目录是`/usr/local/tomcats`，然后再将不同的端口的tomcat放入这个包下，例如80端口的tomcat的文件夹是`/usr/local/tomcats/tomcat80`，依次其它端口类推，这样放入，使得各个端口清晰明了。配置起来方便，我这一共安装了4个tomcat，端口号分别为`80,8080,8081,8082`，如果你还要更多，可以一次类推。

#### 3.2安装tomcat

首先将下载的tomcat放入`/usr/local/tomcats/tomcat80`,
然后解压`tar -zxvf apache-tomcat-7.0.81.tar.gz`,
解压之后，在将解压后的文件移动到`tomcat80`目录下，`mv -r ./apache-tomcat-7.0.81/* ./`
然后再将安装包复制到其它目录下，例如复制到8080端口目录下,`cp -r ./* /usr/local/tomcats/tomcat8080`，依次类推，tomcat就安装好了

#### 3.3配置tomcat全局变量

修改`/etc/profile`文件，`vim /etc/profile`
在末尾加入如下数据

```sh
##########tomcat-80###########
CATALINA_BASE=/usr/local/tomcats/tomcat80
CATALINA_HOME=/usr/local/tomcats/tomcat80
TOMCAT_HOME=/usr/local/tomcats/tomcat80

##########tomcat-8080###########
CATALINA_8080_BASE=/usr/local/tomcats/tomcat8080
CATALINA_8080_HOME=/usr/local/tomcats/tomcat8080
TOMCAT_8080_HOME=/usr/local/tomcats/tomcat8080

##########tomcat-8081###########
CATALINA_8081_BASE=/usr/local/tomcats/tomcat8081
CATALINA_8081_HOME=/usr/local/tomcats/tomcat8081
TOMCAT_8081_HOME=/usr/local/tomcats/tomcat8081

##########tomcat-8082###########
CATALINA_8082_BASE=/usr/local/tomcats/tomcat8082
CATALINA_8082_HOME=/usr/local/tomcats/tomcat8082
TOMCAT_8082_HOME=/usr/local/tomcats/tomcat8082
```

#### 3.4 修改tomcat的bin目录下的文件

修改tomcat安装包下的bin目录下的`cataline.sh`
在`# OS specific support.  $var _must_ be set to either true or false.`下面添加（80端口无需修改这）

```sh
export CATALINA_BASE=$CATALINA_8080_BASE
export CATALINA_HOME=$CATALINA_8080_HOME
```

#### 3.5 修改tomcat的service.xml文件

修改tomcat下的conf下面的`server.xml`

修改服务端口（默认为8005）我这是8080端口，依次加1

```sh
<Server port="8006" shutdown="SHUTDOWN">

```

修改tomcat的端口号

```sh
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

```

修改tomcat连接端口号（默认为8442）我这+1

```sh
<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />

```

依次按照这个方法去修改其他端口号的tomcat

#### 3.6启动测试

依次进入tomcat下的bin目录，启动tomcat `./startup.sh`,，关闭tomcat的命令为`./shutdown.sh`
依次启动并在浏览器中测试，为了方便测试，我建议大家修改下`tomcat/webapp/ROOT`下的`index.jsp`，修改下每个接口的唯一标志，例如我是在tomcat版本后加入了端口号。

### 4. 安装nginx

#### 4.1 配置Nginx的安装环境

安装Nginx需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境，需要安装gcc：`yum install gcc-c++ `，安装完gcc后，才可以进行下一步的安装
#### 4.2 编译安装

##### 4.2.1  解压缩

```sh
将Nginx安装包nginx-1.8.0.tar.gz拷贝至服务器上
解压缩安装包：
`tar -zxvf nginx-1.8.0.tar.gz`
`cd nginx-1.8.0`
```

##### 4.2.2  配置安装参数

参数如下:


~~~sh
```
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
--with-http_ssl_module
```

注：**上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录**
~~~
##### 4.2.3 编译安装

```sh
`make`
`make  install`
```

#### 4.3 启动Nginx

　　`cd /usr/local/nginx/sbin/`
　　`./nginx `
#### 4.4 停止Nginx

`cd /usr/local/nginx/sbin`
`./nginx -s stop`
此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。 
方式2，完整停止(建议使用)：
`cd /usr/local/nginx/sbin`
`./nginx -s quit`
此方式停止步骤是待nginx进程处理任务完毕进行停止。

#### 4.5 重启Nginx

方式1，先停止再启动（建议使用）：
对nginx进行重启相当于先停止nginx再启动nginx，即先执行停止命令再执行启动命令。
如下：
`./nginx -s quit./nginx`

方式2，重新加载配置文件：
当nginx的配置文件nginx.conf修改后，要想让配置生效需要重启nginx，使用-s reload不用先停止nginx再启动nginx即可将配置信息在nginx中生效，如下：
`./nginx -s reload`
#### 4.6 开机自启Nginx

##### 4.6.1 编写shell文件

~~~sh
`vi /etc/init.d/nginx`

```
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
~~~


```
​```sh
![自启命令](https://img-blog.csdn.net/20180409152813251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb2xlYW5pbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```

##### 4.6.2设置文件访问权限

```sh
`chmod a+x /etc/init.d/nginx`   (a+x ==> all user can execute  所有用户可执行)
```

##### 4.6.3 加入到自动列表中

```sh
`vi /etc/rc.local`
 
加入一行  `/etc/init.d/nginx start`    保存并退出，下次重启会生效。
```



### 5.将SSL放置nginx安装目录下

将SSL放置于nginx得安装目录之下，我是直接放在这个目录下得SSL文件夹内

### 6. 配置nginx

配置nginx得配置文件`nginx.conf`

```sh

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
pid /var/run/nginx/nginx.pid;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
    #
	server {
		# 监听端口
		listen       443;
		# 监听域名
		server_name  www.booleandev.xyz;
		# 开启SSL模块
		ssl on;
		# SSL密匙和key
		ssl_certificate      /usr/local/soft/nginx/nginx-1.12.2/ssl/1_booleandev.xyz_bundle.crt;
		ssl_certificate_key  /usr/local/soft/nginx/nginx-1.12.2/ssl/2_booleandev.xyz.key;
		
		# 配置文件
		ssl_session_cache    shared:SSL:1m;
		ssl_session_timeout  5m;
		
		# 加密方式
		ssl_ciphers  HIGH:!aNULL:!MD5;
		ssl_prefer_server_ciphers  on;
		location / {
			# 跳转端口本地8080
			 proxy_pass http://127.0.0.1:8080;  # 内网IP
			 # 携带请求信息
			 proxy_set_header HOST $host;
			 proxy_set_header X-Real-IP $remote_addr;
			 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			 proxy_set_header X-Forwarded-Proto $scheme;
		}
	}
	
	
	server {
		listen       443;
		server_name  blog.booleandev.xyz;
		ssl on;
		ssl_certificate      /usr/local/soft/nginx/nginx-1.12.2/ssl/1_blog.booleandev.xyz_bundle.crt;
		ssl_certificate_key  /usr/local/soft/nginx/nginx-1.12.2/ssl/2_blog.booleandev.xyz.key;

		ssl_session_cache    shared:SSL:1m;
		ssl_session_timeout  5m;

		ssl_ciphers  HIGH:!aNULL:!MD5;
		ssl_prefer_server_ciphers  on;
		location / {
			 proxy_pass http://127.0.0.1:8081;  # 内网IP
			 proxy_set_header HOST $host;
			 proxy_set_header X-Real-IP $remote_addr;
			 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			 proxy_set_header X-Forwarded-Proto $scheme;
		}
	}
	
	# 监听80端口，并且是www开头，则跳转至https://www
	server {
		listen 80;
		server_name www.booleandev.xyz;
		return      301 https://$server_name$request_uri;
	}
	
	# 监听80端口，并且是blog开头，则跳转至https://blog
	server {
		listen 80;
		server_name blog.booleandev.xyz;
		return      301 https://$server_name$request_uri;
	}
	
	# 监听80端口
	server {
		listen 80;
		server_name booleandev.xyz;
		return      301 https://$server_name$request_uri;
	}
	
	#博客
	#server {
	#	listen 80;
	#	server_name blog.booleandev.xyz;
	#	location / {
	#		proxy_pass http://localhost:8081;
	#		proxy_set_header Host $host;
	#		proxy_set_header X-Real-IP $remote_addr;
	#		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	#	}
	#}
	#主页
	#server {
	#	listen 80;
	#	server_name www.booleandev.xyz;
	#	location / {
	#		proxy_pass http://localhost:8080;
	#		proxy_set_header Host $host;
	#		proxy_set_header X-Real-IP $remote_addr;
	#		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	#	}
	#}

	#其余
	#server {
	#	listen 80;
	#	server_name *.booleandev.xyz;
	#	location / {
	#		proxy_pass http://localhost:8081;
	#		proxy_set_header Host $host;
	#		proxy_set_header X-Real-IP $remote_addr;
	#		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	#	}
	#}
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

### 7. 查看成功与否

![1545917185876](E:\workspace\笔记\note\image\2018-12-27-03.png)



> 注:开启https需要开通服务器的443端口



