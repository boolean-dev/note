# linux设置环境变量

- 直接使用目录设置环境变量

  1. 编辑环境变量文件
  `vim /etc/profile`

  1. 在最下一行添加如下文件：
     `export PATH=$PATH:/usr/local/consul`
  2. 设置配置立即生效，否则需要重启后生效
     `source /etc/profile`
- 使用类似于JAVA_HOME编辑
  1. 编辑环境变量文件

     `vim /etc/profile`

  2. 在最下一行添加如下文件

     ```sh
     #下面一行是设置JAVA_HOME
     JAVA_HOME=/usr/local/jdk1.8.0_111;
     #下面一行是设置环境变量
     export PATH=$PATH:$JAVA_HOME/bin
     ```

  3. 设置配置立即生效，否则需要重启后生效

     `source /etc/profile`
