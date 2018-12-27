## 服务器安装多个Tomcat

因为申请了一个域名，然后想设置一个二级域名，让不同的二级域名访问到不同的项目，例如`blog.booleandev.xyz`访问到博客项目，`www.booleandev.xyz`访问到主页，网上找了找资料，发现一般是使用nginx反向代理映射到不同的端口，再跳转到不同的项目，因此想到自己的服务器上安装多个tomcat来玩一玩，顺便这么久没写博客了，最近工作也不是特别忙。好了，废话不说，正文开始了。

### 1.下载tomcat安装包放入服务器中

从网上下载tomcat的linux安装包，然后利用WinSCp软件拖入到服务器，我放得目录是`/usr/local/tomcats`，然后再将不同的端口的tomcat放入这个包下，例如80端口的tomcat的文件夹是`/usr/local/tomcats/tomcat80`，依次其它端口类推，这样放入，使得各个端口清晰明了。配置起来方便，我这一共安装了4个tomcat，端口号分别为`80,8080,8081,8082`，如果你还要更多，可以一次类推。

### 2.安装tomcat
首先将下载的tomcat放入`/usr/local/tomcats/tomcat80`,
然后解压`tar -zxvf apache-tomcat-7.0.81.tar.gz`,
解压之后，在将解压后的文件移动到`tomcat80`目录下，`mv -r ./apache-tomcat-7.0.81/* ./`
然后再将安装包复制到其它目录下，例如复制到8080端口目录下,`cp -r ./* /usr/local/tomcats/tomcat8080`，依次类推，tomcat就安装好了

### 3.配置tomcat全局变量

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

### 4.修改tomcat的bin目录下的文件
修改tomcat安装包下的bin目录下的`cataline.sh`
在`# OS specific support.  $var _must_ be set to either true or false.`下面添加（80端口无需修改这）

```sh
export CATALINA_BASE=$CATALINA_8080_BASE
export CATALINA_HOME=$CATALINA_8080_HOME
```

### 5.修改tomcat的service.xml文件
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

### 6.启动测试
依次进入tomcat下的bin目录，启动tomcat `./startup.sh`,，关闭tomcat的命令为`./shutdown.sh`
依次启动并在浏览器中测试，为了方便测试，我建议大家修改下`tomcat/webapp/ROOT`下的`index.jsp`，修改下每个接口的唯一标志，例如我是在tomcat版本后加入了端口号。

### 7.结束
好了，一个很简单的安装tomcat就完成了，后面如果有时间的话，我再使用nginx完成二级域名到不同项目的设置