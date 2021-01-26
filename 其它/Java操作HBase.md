# Java 操作HBase

## 1. 添加 host 解析

在host文件中，添加解析

```sh
47.114.117.64 sandbox-hdp.hortonworks.com
```

## 2. 下载安装 Hadoop

下载 Hadoop 安装包

https://hadoop.apache.org/releases.html

以管理员身份解压压缩包

## 3. 修改环境变量

添加 Hadoop 的环境变量

```sh
HADOOP_HOME
D:\Program Files\Hadoop\hadoop-3.2.1
```

## 4. 下载

需要下载winutils.exe hadoop.dll等组件放在hadoop安装目录的bin当中。

下载地址：https://github.com/srccodes/hadoop-common-2.2.0-bin

将 winutils.exe 放置于 Hadoop 安装包的 bin 目录下

## 5. 运行程序

```java
public class HBaseConnection {

    public static void main(String[] args) throws IOException {
        //第一步，设置HBsae配置信息
        Configuration configuration = HBaseConfiguration.create();    
        //注意。这里这行目前没有注释掉的，这行和问题3有关系  是要根据自己zookeeper.znode.parent的配置信息进行修改。
       configuration.set("zookeeper.znode.parent","/hbase-unsecure"); //与 hbase-site-xml里面的配置信息 zookeeper.znode.parent 一致  
        configuration.set("hbase.zookeeper.quorum","192.168.8.30");  //hbase 服务地址
        configuration.set("hbase.zookeeper.property.clientPort","2181"); //端口号
        //这里使用的是接口Admin   该接口有一个实现类HBaseAdmin   也可以直接使用这个实现类
        // HBaseAdmin baseAdmin = new HBaseAdmin(configuration);
        Admin admin = ConnectionFactory.createConnection(configuration).getAdmin();
        if(admin !=null){
            try {
                //获取到数据库所有表信息
                HTableDescriptor[] allTable = admin.listTables();
                for (HTableDescriptor hTableDescriptor : allTable) {
                    System.out.println(hTableDescriptor.getNameAsString());
                }
            }catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

