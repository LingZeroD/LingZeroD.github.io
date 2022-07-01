# Docker基本命令

![image-20220628214709364](http://picture.lingzero.cn/202207011957850.png)

## 帮助命令 

```bash
docker version # 显示 Docker 版本信息。 

docker info # 显示 Docker 系统信息，包括镜像和容器数。

docker --help # 帮助
```

## 镜像命令

> 去[docker hub](http://hub.docker.com)，找到镜像

###  docker images

```bash
# 列出本地主机上的镜像
[root@kuangshen ~]# docker images
REPOSITORY TAG IMAGE ID CREATED
SIZE
hello-world latest bf756fb1ae65 4 months ago
13.3kB
# 解释
REPOSITORY 镜像的仓库源
TAG 镜像的标签
IMAGE ID 镜像的ID
CREATED 镜像创建时间
SIZE 镜像大小
# 同一个仓库源可以有多个 TAG，代表这个仓库源的不同版本，我们使用REPOSITORY：TAG 定义不同
的镜像，如果你不定义镜像的标签版本，docker将默认使用 lastest 镜像！
# 可选项
-a： 列出本地所有镜像
-q： 只显示镜像id
--digests： 显示镜像的摘要信息
```

### docker search

```bash
# 搜索镜像
[root@kuangshen ~]# docker search mysql
NAME DESCRIPTION STARS
OFFICIAL
mysql MySQL is a widely used, open-source relation… 9484
[OK]
# docker search 某个镜像的名称 对应DockerHub仓库中的镜像
# 可选项
--filter=stars=50 ： 列出收藏数不小于指定值的镜像。
```

### docker pull

```bash
# 下载镜像
[root@kuangshen ~]# docker pull mysql
Using default tag: latest # 不写tag，默认是latest
latest: Pulling from library/mysql
54fec2fa59d0: Already exists # 分层下载
bcc6c6145912: Already exists
951c3d959c9d: Already exists
05de4d0e206e: Already exists
319f0394ef42: Already exists
d9185034607b: Already exists
013a9c64dadc: Already exists
42f3f7d10903: Pull complete
c4a3851d9207: Pull complete
82a1cc65c182: Pull complete
a0a6b01efa55: Pull complete
bca5ce71f9ea: Pull complete
Digest:
sha256:61a2a33f4b8b4bc93b7b6b9e65e64044aaec594809f818aeffbff69a893d1944 #
签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest # 真实位置
# 指定版本下载
[root@kuangshen ~]# docker pull mysql:5.7
```



### docker rmi

```bash
# 删除镜像
docker rmi -f 镜像id # 删除单个
docker rmi -f 镜像名:tag 镜像名:tag # 删除多个
docker rmi -f $(docker images -qa) # 删除全部
```

## 容器命令

说明：有镜像才能创建容器，我们这里使用 centos 的镜像来测试，就是虚拟一个 centos ！

```bash
docker pull centos
```

### 新建容器并启

```bash
# 命令
docker run [OPTIONS] IMAGE [COMMAND][ARG...]
# 常用参数说明
--name="Name" # 给容器指定一个名字
-d # 后台方式运行容器，并返回容器的id！
-i # 以交互模式运行容器，通过和 -t 一起使用
-t # 给容器重新分配一个终端，通常和 -i 一起使用
-P # 随机端口映射（大写）
-p # 指定端口映射（小结），一般可以有四种写法
ip:hostPort:containerPort
ip::containerPort
hostPort:containerPort (常用)
containerPort
#例如
docker run --name=mynginx   -d  --restart=always -p  88:80   nginx
# 测试
[root@kuangshen ~]# docker images
REPOSITORY TAG IMAGE ID CREATED
SIZE
centos latest 470671670cac 3 months ago
237MB
# 使用centos进行用交互模式启动容器，在容器内执行/bin/bash命令！
[root@kuangshen ~]# docker run -it centos /bin/bash
[root@dc8f24dd06d0 /]# ls # 注意地址，已经切换到容器内部了！
bin etc lib lost+found mnt proc run srv tmp var
dev home lib64 media opt root sbin sys usr
[root@dc8f24dd06d0 /]# exit # 使用 exit 退出容器
exit
[root@kuangshen ~]#
```

### 列出所有运行的容器

```bash
# 命令
docker ps [OPTIONS]
# 常用参数说明
-a # 列出当前所有正在运行的容器 + 历史运行过的容器
-l # 显示最近创建的容器
-n=? # 显示最近n个创建的容器
-q # 静默模式，只显示容器编号。

```

### **退出容器**

```bash
exit # 容器停止退出
ctrl+P+Q # 容器不停止退出
```

### 启动停止容器

```bash
docker start (容器id or 容器名) # 启动容器
docker restart (容器id or 容器名) # 重启容器
docker stop (容器id or 容器名) # 停止容器
docker kill (容器id or 容器名) # 强制停止容器
```

### 删除容器

```bash
docker rm 容器id # 删除指定容器
docker rm -f $(docker ps -a -q) # 删除所有容器
docker ps -a -q|xargs docker rm # 删除所有容器
```

## 常用其他命令

### 后台启动容器

```bash
# 命令
docker run -d 容器名
# 例子
docker run -d centos # 启动centos，使用后台方式启动
# 问题： 使用docker ps 查看，发现容器已经退出了！
# 解释：Docker容器后台运行，就必须有一个前台进程，容器运行的命令如果不是那些一直挂起的命
令，就会自动退出。
# 比如，你运行了nginx服务，但是docker前台没有运行应用，这种情况下，容器启动后，会立即自
杀，因为他觉得没有程序了，所以最好的情况是，将你的应用使用前台进程的方式运行启动。
```

### 查看日志

```bash
# 命令
docker logs -f -t --tail 容器id
# 例子：我们启动 centos，并编写一段脚本来测试玩玩！最后查看日志
[root@kuangshen ~]# docker run -d centos /bin/sh -c "while true;do echo
kuangshen;sleep 1;done"
[root@kuangshen ~]# docker ps
CONTAINER ID IMAGE
c8530dbbe3b4 centos
# -t 显示时间戳
# -f 打印最新的日志
# --tail 数字 显示多少条！
[root@kuangshen ~]# docker logs -tf --tail 10 c8530dbbe3b4
2020-05-11T08:46:40.656901941Z kuangshen
2020-05-11T08:46:41.658765018Z kuangshen
2020-05-11T08:46:42.661015375Z kuangshen
2020-05-11T08:46:43.662865628Z kuangshen
2020-05-11T08:46:44.664571547Z kuangshen
2020-05-11T08:46:45.666718583Z kuangshen
2020-05-11T08:46:46.668556725Z kuangshen
2020-05-11T08:46:47.670424699Z kuangshen
2020-05-11T08:46:48.672324512Z kuangshen
2020-05-11T08:46:49.674092766Z kuangshen
```

### 查看容器/镜像的元数据

```bash
# 命令
docker inspect 容器id
# 测试
[root@kuangshen ~]# docker inspect c8530dbbe3b4
[
{
# 完整的id，有意思啊，这里上面的容器id，就是截取的这个id前几位！
"Id":
"c8530dbbe3b44a0c873f2566442df6543ed653c1319753e34b400efa05f77cf8",
"Created": "2020-05-11T08:43:45.096892382Z",
"Path": "/bin/sh",
"Args": [
"-c",
"while true;do echo kuangshen;sleep 1;done"
],
# 状态
"State": {
"Status": "running",
"Running": true,
"Paused": false,
"Restarting": false,
"OOMKilled": false,
"Dead": false,
"Pid": 27437,
"ExitCode": 0,
"Error": "",
"StartedAt": "2020-05-11T08:43:45.324474622Z",
"FinishedAt": "0001-01-01T00:00:00Z"
},
// ...........
]
```

### 进入正在运行的容器

```bash
# 命令1
docker exec -it 容器id bashShell
# 测试1
[root@kuangshen ~]# docker ps
CONTAINER ID IMAGE COMMAND CREATED
STATUS PORTS NAMES
c8530dbbe3b4 centos "/bin/sh -c 'while t…" 12 minutes
ago Up 12 minutes happy_chaum
[root@kuangshen ~]# docker exec -it c8530dbbe3b4 /bin/bash
[root@c8530dbbe3b4 /]# ps -ef
UID PID PPID C STIME TTY TIME CMD
root 1 0 0 08:43 ? 00:00:00 /bin/sh -c while true;do
echo kuangshen;sleep
root 751 0 0 08:56 pts/0 00:00:00 /bin/bash
root 769 1 0 08:56 ? 00:00:00 /usr/bin/coreutils --
coreutils-prog-shebang=s
root 770 751 0 08:56 pts/0 00:00:00 ps -ef
# 命令2
docker attach 容器id
# 测试2
[root@kuangshen ~]# docker exec -it c8530dbbe3b4 /bin/bash
[root@c8530dbbe3b4 /]# ps -ef
UID PID PPID C STIME TTY TIME CMD
root 1 0 0 08:43 ? 00:00:00 /bin/sh -c while true;do
echo kuangshen;sleep
root 856 0 0 08:57 pts/0 00:00:00 /bin/bash
root 874 1 0 08:57 ? 00:00:00 /usr/bin/coreutils --
coreutils-prog-shebang=s
root 875 856 0 08:57 pts/0 00:00:00 ps -ef
# 区别
# exec 是在容器中打开新的终端，并且可以启动新的进程
# attach 直接进入容器启动命令的终端，不会启动新的进程
```

### 从容器内拷贝文件到主机上

```bash
# 命令
docker cp 容器id:容器内路径 目的主机路径
# 测试
# 容器内执行，创建一个文件测试
[root@c8530dbbe3b4 /]# cd /home
[root@c8530dbbe3b4 home]# touch f1
[root@c8530dbbe3b4 home]# ls
f1
[root@c8530dbbe3b4 home]# exit
exit
# linux复制查看，是否复制成功
[root@kuangshen ~]# docker cp c8530dbbe3b4:/home/f1 /home
[root@kuangshen ~]# cd /home
[root@kuangshen home]# ls
f1
```

## 提交改变

> 将自己修改好的镜像提交

```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
  
docker commit -a "leifengyang"  -m "首页变化" 341d81f7504f guignginx:v1.0

```

> 镜像传输

```bash
# 将镜像保存成压缩包
docker save -o abc.tar guignginx:v1.0

# 别的机器加载这个镜像
docker load -i abc.tar


# 离线安装
```

## 推送远程仓库

> 推送镜像到docker hub；应用市场

```bash
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```

```bash
# 把旧镜像的名字，改成仓库要求的新版名字
docker tag guignginx:v1.0 leifengyang/guignginx:v1.0

# 登录到docker hub
docker login       


docker logout（推送完成镜像后退出）

# 推送
docker push leifengyang/guignginx:v1.0


# 别的机器下载
docker pull leifengyang/guignginx:v1.0
```

