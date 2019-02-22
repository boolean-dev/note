## CenterOs7安装redis

### 1.redis的安装

#### 1.1 下载redis

从官网下载[redis](https://redis.io/download)，并且上传到服务器的安装位置

#### 1.2 安装redis

①解压redis，执行`tar -zxvf  安装包名称`

②make redis源码,`make`

③make install `make install`

### 2. redis的配置

#### 2.1 配置启动脚本

配置redis的启动脚本，放置于`/etc/init.d/`目录下

```sh
#!/bin/sh
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

# redis的端口
REDISPORT=6379
# redis的server地址
EXEC=/usr/local/bin/redis-server
# redis的client地址
CLIEXEC=/usr/local/bin/redis-cli
# redis的密码
PASSWORD=root123456

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
		$CLIEXEC -a $PASSWORD -p $REDISPORT shutdown
#                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac

```



#### 2.2 配置脚本的权限

`chmod +x /etc/init.d/redis`

#### 2.3 配置开机启动

`sudo chkconfig redis on`

### 3. redis的启动和关闭

```sh
# redis的启动
service redis start		#或者 /etc/init.d/redis start
# redis的关闭
service redis stop   		#或者 /etc/init.d/redis stop 
```

