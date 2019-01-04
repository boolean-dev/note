# CenterOS7安装java

```sh
#进入安装目录
cd /usr/local/soft/java

#wget下载java8
#直接进入官网选择相应的版本进行下载，然后把下载链接复制下来就可以下载了
#不时间的下载链接不一样
wget http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz?AuthParam=1536908109_e337304e4470458588ad85166db90d18

#解压缩下载的文件
tar -zxvf jdk-8u181-linux-x64.tar.gz

#编辑环境变量文件
vi /etc/profile

#在文件尾部追加以下文件（JAVA_HOME是你安装java的目录地址）
JAVA_HOME=/usr/local/soft/java/jdk1.8.0_181
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/dt.jar
export JAVA_HOME JRE_HOME PATH CLASSPATH
JAVA_HOME=/usr/local/soft/java/jdk

#刷新环境变量配置文件
source /etc/profile

#查看jdk是否安装成功
java -version
#出现的信息为
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

