## BAT批处理运行项目

### 1. 批处理简介

顾名思义，批处理文件是将一系列命令按一定的顺序集合为一个可执行的文本文件，其扩展名为BAT或者CMD。这些命令统称批处理命令。

### 2. 编写启动zookeeper脚本

```sh
@echo off
call "D:\Program Files\zookeeper-3.4.10\bin\zkServer.cmd"
```

### 3. 编写启动kafka脚本

```sh
@echo off
call "D:\Program Files\kafka_2.11-1.1.1\bin\windows\kafka-server-start.bat" "D:\Program Files\kafka_2.11-1.1.1\config\server.properties"
```

### 4. 编写启动consul脚本

```sh
#开启一个窗口启动consul脚本，并且在5S后，执行一段node代码
start consul.exe agent -dev -client 0.0.0.0 -ui
E:
cd E:\workspace\onegene-demo\bto-service-test\
choice /T 5 /C ync /CS /D y /n
node consul -import
```

### 5. 编写启动java的jar包脚本

```sh
#编写启动jar包的一个脚本
#进入本地maven仓库，然后设置java虚拟机参数启动jar包
@echo off
title gateway
D:
cd D:\Program Files\Apache Software Foundation\MavenRepository\com\utoper\bto\gateway\bto-gateway\3.0.0-SNAPSHOT\
java -jar -Xms128M -Xmx256M .\bto-gateway-3.0.0-SNAPSHOT.jar
```

