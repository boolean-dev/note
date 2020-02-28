# Jenkins集成部署SpringBoot

## 1. 前言

随着业务的增长，需求也开始增多，每个需求的大小，开发周期，发布时间都不一致。基于微服务的系统架构，功能的叠加，对应的服务的数量也在增加，大小功能的快速迭代，更加要求部署的快速化，智能化。因此，传统的人工部署已经心有余而力不足。
持续集成，持续部署，持续交互对于微服务开发来说，是提高团队整体效率不可或缺的一环。合理的使用CI,CD能够极大的提高了生产效率，也提高了产品的交互质量。

本文主要介绍的内容有如下几点

- docker安装Jenkins
- Jenkins安装后的初始化和相关配置
- SpringBoot的docker打包镜像
- Jenkins从Github拉取源码
- Jenkins将SpringBoot项目部署成docker

> 阅读本文需要对docker和linux有一定的了解

## 2. Jenkins安装及配置

### 2.1 安装Jenkins

Jenkins的安装采用的是docker版本安装，相对与其他版本的安装，会简单很多

运行Jenkins容器命令如下

```shell
docker run \
  --privileged=true \
  --name jenkins \
  -d \
  -u root \
  -p 8080:8080 \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /usr/local/soft/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e TZ=Asia/Shanghai \
  -v "$HOME":/home \
  jenkins/jenkins
```

相关参数说明

- --privileged：使得容器内的用户拥有root权限
- --name：运行容器的名称
- -d： 容器后台运行
- -u：指定容器的用户
- -v /usr/bin/docker:/usr/bin/docker：将宿主的/var/run/docker.sock文件挂载到容器内的同样位置，从而让容器内可以通过unix socket调用宿主的Docker引擎
-  -v /usr/bin/docker:/usr/bin/docker：同上
- -v /usr/local/soft/jenkins:/var/jenkins_home：将Jenkins的数据目录挂载到主机目录/usr/local/soft/jenkins
- -e：指定时区

### 2.2 初始化Jenkins

1. 浏览器打开Jenkins地址`(http://127.0.0.1/):8080`

![image1.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image1-658060b8.png)

找到Jenkins映射目录，找到相应文件，查看密码，登录

2. 安装插件，一般直接选择推荐的插件安装

![image2.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image2-2611a044.png)

3. 安装插件较慢，等待片刻即可

![image3.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image3-94cfb863.png)

4. 创建系统管理员

![image4.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image4-23bf8e9d.png)

5. 安装完成，进入下一步

![image5.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image5-995f4066.png)

### 2.3 配置相应的Jenkins

#### 2.3.1 maven

1. 安装maven

因为使用Jenkins默认的maven下载包比较慢，所以这里我自行安装maven

进入设置->全局工具设置

![image6.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image6-912ce2ac.png)

这里我安装的是maven3.6.1

2. 修改maven的配置

修改Jenkins安装目录下的maven配置，我的maven在主机的目录为`/usr/local/soft/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.1`，自己可自行判断目录

配置文件修改后为

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>/var/jenkins_home/tools/hudson.tasks.Maven_MavenInstallation/MavenRepository</localRepository>

  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <pluginGroup>com.spotify</pluginGroup>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <!-- mirrors
   | This is a list of mirrors to be used in downloading artifacts from remote repositories.
   |
   | It works like this: a POM may declare a repository to use in resolving certain artifacts.
   | However, this repository may have problems with heavy traffic at times, so people have mirrored
   | it to several places.
   |
   | That repository definition will have a unique id, so we can create a mirror reference for that
   | repository, to be used as an alternate download site. The mirror site will be the preferred
   | server for that repository.
   |-->
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <mirrorOf>central</mirrorOf>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
  </mirrors>

  <!-- profiles
   | This is a list of profiles which can be activated in a variety of ways, and which can modify
   | the build process. Profiles provided in the settings.xml are intended to provide local machine-
   | specific paths and repository locations which allow the build to work in the local environment.
   |
   | For example, if you have an integration testing plugin - like cactus - that needs to know where
   | your Tomcat instance is installed, you can provide a variable here such that the variable is
   | dereferenced during the build process to configure the cactus plugin.
   |
   | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
   | section of this document (settings.xml) - will be discussed later. Another way essentially
   | relies on the detection of a system property, either matching a particular value for the property,
   | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
   | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
   | Finally, the list of active profiles can be specified directly from the command line.
   |
   | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
   |       repositories, plugin repositories, and free-form properties to be used as configuration
   |       variables for plugins in the POM.
   |
   |-->
  <profiles>
    <!-- profile
     | Specifies a set of introductions to the build process, to be activated using one or more of the
     | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
     | or the command line, profiles have to have an ID that is unique.
     |
     | An encouraged best practice for profile identification is to use a consistent naming convention
     | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
     | This will make it more intuitive to understand what the set of introduced profiles is attempting
     | to accomplish, particularly when you only have a list of profile id's for debug.
     |
     | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | Here is another profile, activated by the system property 'target-env' with a value of 'dev',
     | which provides a specific path to the Tomcat instance. To use this, your plugin configuration
     | might hypothetically look like:
     |
     | ...
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
     |
     | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
     |       anything, you could just leave off the <value/> inside the activation-property.
     |
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>

  <!-- activeProfiles
   | List of profiles that are active for all builds.
   |
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>

```



> 修改的位置分别为：
>
> 1. maven本地仓库的位置：
>
>  ```xml
> <localRepository>/var/jenkins_home/tools/hudson.tasks.Maven_MavenInstallation/MavenRepository</localRepository>
>  ```
>
> 2. mvn打包docker的插件，不添加会报错
>
>  ```xml
>  <pluginGroup>com.spotify</pluginGroup>
>  ```
>
> 3. 阿里云镜像加速
>
> ```xml
>  <mirror>
>    <id>alimaven</id>
>    <name>aliyun maven</name>
>    <mirrorOf>central</mirrorOf>
>    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
>  </mirror>
> ```

#### 2.3.2 配置环境变量

系统管理->系统设置

添加环境变量

![image7.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image7-24993d55.png)

添加的环境变量为:

```sh
MAVEN_HOME
/var/jenkins_home/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.1
Path
$Path:$MAVEN_HOME/bin:
```

## 2. SpringBoot项目配置

### 3.1 Dockerfile文件编写

```dockerfile
FROM java:latest

ENV SERVER_PORT 8080
ENV CONSUL_SERVER 127.0.0.1
ENV JVM_MEMORY 512M

COPY ./target/*.jar /tmp

RUN cp -f ./tmp/*.jar /app.jar

EXPOSE 8080

CMD echo "The application is starting..." && \
    java -Xmx${JVM_MEMORY} -jar /app.jar
```

### 3.2 pom文件编写

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.tao</groupId>
	<artifactId>jenkins</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>jenkins</name>
	<description>Demo project for Spring Boot</description>

	<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
        <docker.image.prefix>SpringBoot-jenkins</docker.image.prefix>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<repositories>
		<repository>
			<id>aliyun-repos</id>
			<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.11</version>
                <configuration>
					<imageName>mytest/${project.artifactId}</imageName>
					<dockerDirectory>./</dockerDirectory>
					<resources>
						<resource>
							<targetPath>/</targetPath>
							<directory>${project.build.directory}</directory>
							<include>${project.build.finalName}.jar</include>
						</resource>
					</resources>
                </configuration>
            </plugin>
        </plugins>
	</build>

</project>

```

### 3.3 项目github地址

[Github地址](https://github.com/boolean-dev/jenkins.git)

`https://github.com/boolean-dev/jenkins.git`

## 3. Jenkins部署

### 3.1 源码管理

填写Github的项目地址即可

![image8.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image8-8a9e26f5.png)

### 3.2 项目构建参数

1. maven打包

选择maven的版本，并且执行maven构建命令

`clean install -Dmaven.test.skip=true`

2. 执行shell

    ```shell
    mvn docker:build
    echo "which docker"
    docker -v
    echo "当前docker 镜像："
    echo "启动容器----->"
    docker images 
    docker run --name springboot -p 8080:8080 -d mytest/jenkins
    echo "启动服务成功！"
    ```

    

![image9.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image9-dbd4d25b.png)

> 注：第一次执行shell的时候，因为springboot容器还未运行，所以无需停止和移除。所以第二次构建容器的时候，运行前需提前停止和移除容器，所以采用以下的shell
>
> ```shell
> mvn docker:build
> echo "which docker"
> docker -v
> echo "当前docker 镜像："
> echo "启动容器----->"
> docker stop springboot
> echo "移除旧容器"
> docker rm springboot
> docker images 
> docker run --name springboot -p 8080:8080 -d mytest/jenkins
> echo "启动服务成功！"
> ```

### 3.3 构建项目

可能在构建的过程中还会存在部分bug，只需查看相应日志，再google，一般都能解决。

构建完成后，在宿主机上执行`docker ps`查看容器是否运行

![image11.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image11-8c2904cd.png)

如图，部署成功了

![image10.png](https://boolean-dev.oss-cn-hangzhou.aliyuncs.com/blog/image/2020-02/image10-cffb51cd.png)

## 4. 后记

本篇博客实现了简单的SpringBoot、jenkins和docker继承部署，基本上全是运行在docker容器中，由此可见`docker`确实十分强大。

但是本篇博客之讲了简单的集成部署，并且采用的是自由式风格的编排，现主流的编排是使用流水线，那样更加便捷，并且功能更加强大。

现阶段还存在的缺点：

- 使用自由式风格以及shell的编排模块，部署大量项目时繁杂
- 本次部署是Jenkins部署项目是在宿主机上，具有很大的局限性
- 历史镜像尚未保存
- 不能够多分枝部署
- 未实现github提交代码，自动更新
- 镜像存储在本地，未存储在阿里云等镜像仓库中

后期可能进行的优化：

- 镜像存储在阿里云镜像仓库中，并且根据git的tag对镜像进行备份
- 多分枝集成部署
- 实现github钩子程序
- docker运行采用swarm集群或者K8s集群，便于管理集群
- ...

## 5. 参考链接

[使用Jenkins和Docker持续集成java项目](https://www.chenb.top/2018/05/14/spring-boot-admin-jenkins-docker/)

[解放双手 | Jenkins + gitlab + maven 自动打包部署项目](https://zhuanlan.zhihu.com/p/54973160)

[使用 Jenkins 自动部署 Docker 服务（一、Jenkins 搭建篇）](https://segmentfault.com/a/1190000016051537)

[使用 Jenkins 自动部署 Docker 服务（二、构建部署篇）](https://segmentfault.com/a/1190000016051754)

[基于Jenkins，docker实现自动化部署（持续交互）](https://www.cnblogs.com/bigben0123/p/7886092.html)

[CentOS7中Docker文件挂载，容器中的权限问题](https://www.iteye.com/blog/panyongzheng-2242442)

[SpringBoot | 第十四章：基于Docker的简单部署](https://blog.lqdev.cn/2018/07/27/springboot/chapter-fourteen/)