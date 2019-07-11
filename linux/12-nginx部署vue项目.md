# nginx部署vue项目

## 1. 前言

此文档主要介绍如何使用`nginx`部署`vue`等前端项目，并配置SSL证书部署的前提下是服务器已经安装`nginx`，前端项目已打包成静态文件

## 2. 部署过程

### 2.1 申请SSL证书

向服务商（阿里云）申请SSL证书，并且下载`nginx`版本的key和密匙，放置于nginx的安装目录之下

### 2.2 修改nginx配置文件

修改nginx的配置文件`nginx.cnf`，修改的内容如下

```sh
user www-data;
worker_processes 4;
pid /run/nginx.pid;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_disable "msie6";

	##
	# Virtual Host Configs
	##

    #设定虚拟主机配置
  server {
        #侦听443端口https
        listen    443;
        #定义使用 www.nginx.cn访问
        server_name  www.nginx.cn;

      # SSL证书配置
      ssl on;
      # 证书路径
      ssl_certificate      /etc/nginx/ssl/www.nginx.cn.pem;
      ssl_certificate_key  /etc/nginx/ssl/www.nginx.cn.key;

      ssl_session_cache    shared:SSL:1m;
      ssl_session_timeout  5m;

      ssl_ciphers  HIGH:!aNULL:!MD5;
      ssl_prefer_server_ciphers  on;

        # vue打包后静态文件存储路径
        root /home/wwwroot/www.nginx.cn/;

        #默认请求
        location / {
        		# 除去www.nginx.cn/路径的其它路径访问路径例如www.nginx.cn/user
						try_files $uri $uri/ /index.html last;
            #定义首页索引文件的名称
            index index.html index.htm;   

        }
        # 接口请求转发 www.nginx.cn/api/后面的请求，转发到本地8080端口
        location ^~ /api/ {
            proxy_pass  http://127.0.0.1:8080;
        }
        # 定义错误提示页面
        error_page   500 502 503 504 /50x.html;
        location = /50x.html {
        }

        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {

            #过期30天，静态文件不怎么更新，过期可以设大一点，
            #如果频繁更新，则可以设置得小一点。
            expires 30d;
        }
    }
	
	# http转https http80端口重定向至443端口
	server {
		listen 80;
		server_name m.1gene.com.cn;
		return      301 https://$server_name$request_uri;
	}
}
```

### 2.3 重启nginx

```sh
service nginx restart
```