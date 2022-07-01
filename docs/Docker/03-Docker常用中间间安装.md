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

