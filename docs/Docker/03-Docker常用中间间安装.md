# 可视化安装

> Portainer（先用这个）

```bash
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock \
--privileged=true portainer/portainer
```

介绍： Portainer是Docker的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷 的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和 服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管 理的全部需求。 如果仅有一个docker宿主机，则可使用单机版运行，Portainer单机版运行十分简单，只需要一条语句即 可启动容器，来管理该机器上的docker镜像、容器等数据。

> Rancher（CI/CD再用这个）

```bash
#安装rancher-server
docker run --name rancher-server -p 8000:8080 -v
/etc/localtime:/etc/localtime:ro -d rancher/server
#安装agent
docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v
/var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.11
http://39.101.191.131:8000/v1/scripts/D3DBD43F263109BB881F:1577750400000:7M0y
BzCw4XSxJklD7TpysYIpI
```



# 部署中间件

## Redis

```bash
docker pull redis:6.2.4
 
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]


#redis使用自定义配置文件启动
docker run -v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data \
-d --name myredis \
-p 6379:6379 \
redis:6.2.4  redis-server /etc/redis/redis.conf --appendonly yes




```

> 需要先在宿主机上创建 `redis.conf`不然会变成文件夹

## MySQL

```bash
# 1、搜索镜像
[root@kuangshen ~]# docker search mysql
NAME DESCRIPTION
STARS
mysql MySQL is a widely used, open-source
relation… 9488
# 2、拉取镜像
[root@kuangshen ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
54fec2fa59d0: Already exists
bcc6c6145912: Pull complete
951c3d959c9d: Pull complete
05de4d0e206e: Pull complete
319f0394ef42: Pull complete
d9185034607b: Pull complete
013a9c64dadc: Pull complete
e745b3361626: Pull complete
03145d87b451: Pull complete
3991a6b182ee: Pull complete
62335de06f7d: Pull complete
Digest:
sha256:e821ca8cc7a44d354486f30c6a193ec6b70a4eed8c8362aeede4e9b8d74b8ebb
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
# 3、启动容器 -e 环境变量！

# 在本地创建mysql的映射目录
mkdir -p /root/mysql/data /root/mysql/logs /root/mysql/conf
# 在/root/mysql/conf中创建 *.cnf 文件(叫什么都行)
touch my.cnf

# 注意： mysql的数据应该不丢失！先体验下 -v 挂载卷！ 参考官方文档
docker run -p 3306:3306 --name mysql \
-v /root/mysql/conf:/etc/mysql/conf.d \
-v /root/mysql/logs:/logs \
-v /root/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root -d mysql:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
# 4、使用本地的sqlyog连接测试一下 3306

```



## tomcat

```bash
# 1、下载tomcat镜像
docker pull tomcat
# 2、启动
docker run -d -p 8080:8080 --name tomcat9 tomcat
# 3、进入tomcat
docker exec -it tomcat9 /bin/bash

```

## Mongo

```bash
//创建并运行容器，开启身份认证
docker run -d -p 27017:27017 -v mongo_configdb:/data/configdb -v mongo_db:/data/db --name mongo mongo: --auth

docker exec -it mongo mongo admin
//创建管理员
db.createUser({ user: 'ling', pwd: 'ling@123', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ,"readWriteAnyDatabase"] });

//创建普通用户
//1.admin 授权
db.auth("ling","ling@123");
//2.却换 test库（不存在自动创建）
use test
//3.创建test库下的用户
db.createUser({ user: 'emos', pwd: '123456', roles: [{ role: "readWrite", db: "emos" }] });

```

## RabbitMQ

```bash
docker run \
 -e RABBITMQ_DEFAULT_USER=ling \
 -e RABBITMQ_DEFAULT_PASS=ling@123 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```

- -d 后台运行
- -p 隐射端口
- –name 指定rabbitMQ名称
- RABBITMQ_DEFAULT_USER 指定用户账号
- RABBITMQ_DEFAULT_PASS 指定账号密码

执行如上命令后访问：http://ip:15672/

## Nacos

单机版

```bash
docker pull nacos/nacos-server:1.3.1

mkdir -p /home/nacos/logs/
mkdir -p /home/nacos/init.d/

vim /home/nacos/init.d/custom.properties
#添加一下数据，db.url，db.user，db.pass改为你自己的mysql的信息
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848

spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=root

nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false

management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false

server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i

nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
#运行镜像
docker  run \
--name nacos -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /home/nacos/logs:/home/nacos/logs \
-v /home/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties \
nacos/nacos-server

#访问地址:http://域名或ip地址:8848/nacos 账号:nacos 密码:nacos
```



## FASTDFS

```bash
docker pull delron/fastdfs
# 创建挂载目录(已经创建吗，删除后重新创建，避免jia'zai)
mkdir -p /usr/local/server/fastdfs/tracker/data
mkdir -p /usr/local/server/fastdfs/storage/data
mkdir -p /usr/local/server/fastdfs/storage/path
#创建tracker容器（跟踪服务器容器）
docker run -d --name tracker -p 22122:22122 -v /usr/local/server/fastdfs/tracker/data:/var/fdfs delron/fastdfs tracker

#创建storage容器 8888为Nginx对应的访问端口，23000是storage服务端口
docker run -d --name storage -p 8888:8888 -p 23000:23000 -e TRACKER_SERVER=192.168.0.131:22122 -v /usr/local/server/fastdfs/storage/data:/var/fdfs -e GROUP_NAME=group1 delron/fastdfs storage

#client测试
docker exec -it storage bash
cd /var/fdfs
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf  <local_filename>


```

## Drools workbench kie-server

https://blog.csdn.net/zhang907743237/article/details/103959531

```bash

#workbench 的安装
docker pull jboss/drools-workbench-showcase
docker run -p 8080:8080 -p 8001:8001 -d --name drools-workbench jboss/drools-workbench-showcase:latest
#账号密码
USER        PASSWORD    ROLE
*********************************************
admin       admin       admin,analyst,kiemgmt
krisv       krisv       admin,analyst
john        john        analyst,Accounting,PM
sales-rep   sales-rep   analyst,sales
katy        katy        analyst,HR
jack        jack        analyst,IT

#kie-server安装
docker pull jboss/kie-server-showcase
docker run -p 8180:8080 -d --name kie-server --link drools-workbench:kie_wb jboss/kie-server-showcase:latest

```

## zookeeper

### 单机

```bash
#启动容器：
docker run -id --name my_zookeeper -p 2181:2181 zookeeper:3.4.14
#查看容器运行情况：
docker logs -f my_zookeeper
#连接zookeeper
docker run -it --rm --link my_zookeeper:zk zookeeper:3.4.14 zkCli.sh -server zk
```

### 集群

**Docker Compose安装：**

下载 Docker Compose 的当前稳定版本：

```bash
#安装pip
yum -y install epel-release python-pip
#升级pip
pip install --upgrade pip 
pip install --upgrade setuptools --upgrade requests
#查看pip版本
pip -V
#使用pip安装Docker Compose
pip install docker-compose
```

测试是否安装成功：

```bash
docker-compose --version
```

删除docker-compose

```bash
pip uninstall docker-compose -y
```

首先创建一个名为 **docker-compose.yml** 的文件：

```bash
mkdir ~/zk_cluster
cd ~/zk_cluster
vi docker-compose.yml
```

docker-compose.yml 内容如下:

```bash
version: '3.8'
services:
    zk01:
        image: zookeeper:3.4.14
        restart: always
        container_name: zk01
        ports:
            - "2181:2181"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zk01:2888:3888 server.2=zk02:2888:3888 server.3=zk03:2888:3888
 
    zk02:
        image: zookeeper:3.4.14
        restart: always
        container_name: zk02
        ports:
            - "2182:2181"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zk01:2888:3888 server.2=zk02:2888:3888 server.3=zk03:2888:3888
 
    zk03:
        image: zookeeper:3.4.14
        restart: always
        container_name: zk03
        ports:
            - "2183:2181"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zk01:2888:3888 server.2=zk02:2888:3888 server.3=zk03:2888:3888
```

注意Compose文件的版本，需要符合以下要求：

| Compose file format | Docker Engine release |
| :-----------------: | :-------------------: |
|         3.8         |       19.03.0+        |
|         3.7         |       18.06.0+        |
|         3.6         |       18.02.0+        |
|         3.5         |       17.12.0+        |
|         3.4         |       17.09.0+        |
|         3.3         |       17.06.0+        |
|         3.2         |       17.04.0+        |
|         3.1         |        1.13.1+        |
|          3          |        1.13.0+        |
|         2.4         |       17.12.0+        |
|         2.3         |       17.06.0+        |
|         2.2         |        1.13.0+        |
|         2.1         |        1.12.0+        |
|          2          |        1.10.0+        |
|          1          |        1.9.1.+        |

在 docker-compose.yml 当前目录下运行命令：

```bash
#校验docker-compose.yml
docker-compose config
#启动zookeeper集群
COMPOSE_PROJECT_NAME=zk_otter docker-compose up -d
```

查看启动的 ZK 容器，运行以下命令：

```
COMPOSE_PROJECT_NAME=zk_otter docker-compose ps
```

![1590570683558](C:/Windows/system32/img/1590570683558.png)

*COMPOSE_PROJECT_NAME=zk_test 这个环境变量, 这是为我们的 compose 工程起一个名字, 以免与其他的 compose 混淆.*

依次执行命令：

```bash
docker exec zk01 /bin/bash -c 'bin/zkServer.sh status'
docker exec zk02 /bin/bash -c 'bin/zkServer.sh status'
docker exec zk03 /bin/bash -c 'bin/zkServer.sh status'
```

可以看到一个主，两个从，集群搭建完成

集群连接：

查看Networks名称

```bash
docker inspect -f '{{.NetworkSettings.Networks}}' zk01
```

根据Networks名称连接集群：

```bash
docker run -it --rm \
        --link zk01:zk1 \
        --link zk02:zk2 \
        --link zk03:zk3 \
        --net zk_otter_default \
        zookeeper:3.4.14 zkCli.sh -server zk1:2181,zk2:2181,zk3:2181
```

##  kafka

```bash
#拉取镜像
docker pull wurstmeister/kafka
#启动容器
docker run -id --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=68.79.63.42:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://68.79.63.42:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka

#查看容器运行情况：
docker logs -f kafka
```

- 参数说明

```bash
-e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己
-e KAFKA_ZOOKEEPER_CONNECT=68.79.63.42:2181 配置zookeeper管理kafka的路径
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://68.79.63.42:9092  把kafka的地址端口注册给zookeeper
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口
-v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间
```

- 验证

安装完成后需要测试一下安装是否成功：

```bash
#进入kafka容器
docker exec -it kafka /bin/bash
#进入bin目录
cd /opt/kafka_2.12-2.5.0/bin/
#执行命令，创建名称为test的队列，此队列有一个副本，一个分区
./kafka-topics.sh --create --zookeeper 68.79.63.42:2181 --replication-factor 1 --partitions 1 --topic test
#查看刚刚创建的队列
./kafka-topics.sh -list -zookeeper 68.79.63.42:2181
```

- 消息发送和接收

接下来通过kafka提供的脚本测试发送消息和接收消息：

```bash
#执行命令，启动消费端，监听test队列
./kafka-console-consumer.sh --bootstrap-server 68.79.63.42:9092 --topic test --from-beginning
#新开一个命令窗口，启动生产者，向test队列发送消息
./kafka-console-producer.sh --broker-list 68.79.63.42:9092 --topic test
#测试完成后按Ctrl+C退出
```
