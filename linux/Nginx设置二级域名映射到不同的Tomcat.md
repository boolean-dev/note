## 一、前言
在之前的博客中，已经安装好了多个tomcat和nginx，本篇博客将介绍如何设置不同的二级域名转发到不同的tomcat上

## 二 、配置服务器端

我使用的是腾讯云服务器，只需要在云解析中配置相关域名信息即可

![云解析配置](https://img-blog.csdn.net/20180409153951221?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb2xlYW5pbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 三、配置nginx
进入nginx的配置文件中
`cd /usr/local/nginx/conf`
`vim nginx.conf`

```sh
server {
listen 80;
server_name yjt.booleandev.xyz;
location / {
proxy_pass http://localhost:8081;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}

server {
listen 80;
server_name zlj.booleandev.xyz;
location / {
proxy_pass http://localhost:8082;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}

server {
listen 80;
server_name *.booleandev.xyz;
location / {
proxy_pass http://localhost:8080;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}

```
## 四、重新加载配置文件
`./nginx -s reload`