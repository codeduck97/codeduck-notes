本笔记整理自：https://www.cssjs.cn/doc/3/21.html

# Images镜像命令

## 获取镜像列表

```bash
# 列出本地主机上的镜像
$ docker images [OPTIONS] [REPOSITORY[:TAG]]

    -a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；

    --digests :显示镜像的摘要信息；

    -f :显示满足条件的镜像；

    --format :指定返回值的模板文件；

    --no-trunc :显示完整的镜像信息；

    -q :只显示镜像ID。

```

## 搜索镜像

```shell
# 搜索镜像
$ docker search [OPTIONS] TERM

# 可选参数
    --automated：弃用，只列出自动构建类型的镜像

    --filter , -f：基于给定条件过滤输出

    --format：使用模板格式化显示输出    

    --limit：Max number of search results ，默认值25

    --no-trunc：禁止截断输出

    --stars , -s：弃用，只显示收藏数不小于几颗星的镜像，移到--filter中使用 

# --filter=stars=600: 只显示 starts>=600 的mysql镜像
$ docker search --filter=stars=600 mysql
```

## 拖取镜像

```bash
$ docker pull [OPTIONS] NAME[:TAG|@DIGEST]
    --all-tags , -a：下载所有标签镜像

    --disable-content-trust：跳过验证，默认值true

    --platform：体验版(daemon)API 1.32+ 多平台时进行选择    

    --quiet , -q：静默输出    
    
# 拖取最新的版本镜像
$ docker pull httpd

# 拖取指定版本镜像
$ docker pull ubuntu:13.10
```

## 删除镜像

可以使用镜像名称、镜像短id、长id、标签、digest来移除镜像。如果一个镜像有其他镜像引用，你必须先移除其它镜像才能移除当前镜像

```bash
$ docker rmi [OPTIONS] IMAGE [IMAGE...]

# 可选参数
    --force , -f：强制移除镜像

    --no-prune：不移除该镜像的过程镜像，默认移除   

# 删除指定镜像
$ docker rmi hello-world

# 强制删除指定镜像
$ docker rmi -f mysql

#多个镜像删除，不同镜像间以空格间隔
$ docker rmi -f mysql tomcat nginx
```

# 容器命令

## 创建容器不启动

```bash
$ docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

# 可选参数
    -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
    -d: 后台运行容器，并返回容器ID；
    -i: 以交互模式运行容器，通常与 -t 同时使用；
    -P: 随机端口映射，容器内部端口随机映射到主机的端口
    -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
    -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
    --name="nginx-lb": 为容器指定一个名称；
    --dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
    --dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
    -h "mars": 指定容器的hostname；
    -e username="ritchie": 设置环境变量；
    --env-file=[]: 从指定文件读入环境变量；
    --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；
    -m :设置容器使用内存最大值；
    --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
    --link=[]: 添加链接到另一个容器；
    --expose=[]: 开放一个端口或一组端口；
    --volume , -v: 绑定一个卷
    
 # 创建启动容器
 $ docker create -t -i fedora bash
 
 # 绑定卷
 $ docker create -v /data --name data ubuntu
```

## 创建新容器并启动

```bash
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# 可选参数
    -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
    -d: 后台运行容器，并返回容器ID；
    -i: 以交互模式运行容器，通常与 -t 同时使用；
    -P: 随机端口映射，容器内部端口随机映射到主机的端口
    -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
    -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
    --name="nginx-lb": 为容器指定一个名称；
    --dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
    --dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
    -h "mars": 指定容器的hostname；
    -e username="ritchie": 设置环境变量；
    --env-file=[]: 从指定文件读入环境变量；
    --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；
    -m :设置容器使用内存最大值；
    --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
    --link=[]: 添加链接到另一个容器；
    --expose=[]: 开放一个端口或一组端口；
    --volume , -v: 绑定一个卷

# 使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。
$ docker run --name mynginx -d nginx:latest

# 使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。
docker run -P -d nginx:latest

# 使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。
$ docker run -p 80:80 -v /data:/data -d nginx:latest

# 绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。
$ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash

# 使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/#
```



## 列出容器

容器的STATUS有7种:

1. created（已创建）
2. restarting（重启中）
3. running（运行中）
4. removing（迁移中）
5. paused（暂停）
6. exited（停止）
7. dead（死亡）

```bash
$ docker ps [OPTIONS]

# 可选参数
    -a :显示所有的容器，包括未运行的。

    -f :根据条件过滤显示的内容。

    --format :指定返回值的模板文件。

    -l :显示最近创建的容器。

    -n :列出最近创建的n个容器。

    --no-trunc :不截断输出。

    -q :静默模式，只显示容器编号。

    -s :显示总的文件大小。
    
# 列出最近创建的5个容器
$ docker ps -n 5
```

## docker exec 连接运行容器

```bash
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

# 可选参数
    -d :分离模式: 在后台运行

    -i :即使没有附加也保持STDIN 打开

    -t :分配一个伪终端
# 在容器 mynginx 中开启一个交互模式的终端:
$ docker exec -i -t  mynginx /bin/bash
$ root@b1a0703e41e7:/#


# 通过 exec 命令对指定的容器执行 bash:
$ docker exec -it 9df70f9a0714 /bin/bash

# 在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本:
$ docker exec -it mynginx /bin/sh /root/runoob.sh

# 退出交互终端
$ exit
```

## docker attach 连接运行容器

要attach上去的容器必须正在运行，可以同时连接上同一个container来共享屏幕（与screen命令的attach类似），也就是说当我们打开两个窗口用attach进入同一个容器的时候，两个窗口显示的内容是一致的。

当我们输入exit来退出容器时，容器会被关闭，当我们直接关系shell窗口，容器则不会被关闭，这也就导致操作上的一些麻烦，所以没有特殊的要求，建义还是使用 [docker exec](https://www.cssjs.cn/doc/3/28.html) 来操作容器。

官方文档中说attach后可以通过CTRL-C来detach，但实际上经过我的测试，如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。这不是我们想要的，detach的意思按理应该是脱离容器终端，但容器依然运行。好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。

```bash
# 连接到正在运行中的容器
$ docker attach [OPTIONS] CONTAINER

# 可选参数
    --detach-keys    覆写脱离容器的快捷键

    --no-stdin    不打开输入功能

    --sig-proxy    代理所有进程所接收的信号，默认true 
    
$ docker attach --sig-proxy=false mynginx


# 退出正在运行的容器
$ exit		 	# 直接停止容器并退出
$ Ctrl + P + Q 	# 容器后台运行并退出
```

## 杀掉运行中的容器

```bash
$ docker kill [OPTIONS] CONTAINER [CONTAINER...]

# 可选参数
--signal , -s :向容器发送一个信号，默认值KILL

# 自定义发送信息
$ docker kill --signal=SIGHUP  my_container
```

## 停止容器

```bash
$ docker stop [OPTIONS] CONTAINER [CONTAINER...]

# 可选参数
    --time , -t：等待容器停止秒数，默认10秒    

$ docker stop my_container

# 停止所有容器
$ docker stop $(docker ps -aq)
```

## 启动容器

```bash
$ docker start [OPTIONS] CONTAINER [CONTAINER...]

# 可选参数
    --attach , -a：打开输入输出并发送信号   

    --checkpoint：体验版 (daemon) 从检查点重置

    --checkpoint-dir：体验版 (daemon) 使用自定义检查点目录   

    --detach-keys：覆盖容器后台运行的一些参数信息

    --interactive , -i：打开容器交互  
```

## 暂停容器

```bash
$ docker pause CONTAINER [CONTAINER...]

# 暂停
$ docker pause my_container
```

## 删除容器

```bash
$ docker rm [OPTIONS] CONTAINER [CONTAINER...]

# 可选参数
    --force , -f：通过 SIGKILL 信号强制删除一个运行中的容器。

    --link , -l：移除容器间的网络连接，而非容器本身

    --volumes , -v：删除与容器关联的卷
    
# 移除Redis容器
$ docker rm redis

# 移除容器对应连接
$ docker rm --link /webapp/redis

# 强制移除运行中的容器
$ docker rm --force redis

# 删除所有停止的容器
$ docker rm $(docker ps -a -q)

# 移除容器和它的卷
$ docker rm -v redis

# 移除容器并选择性移除卷
$ docker create -v awesome:/foo -v /bar --name hello redis
```

## 重启容器

```bash
docker restart [OPTIONS] CONTAINER [CONTAINER...]

# 可选参数
--time , -t：重启容器之前等待秒数。默认值10秒
```

## 重命名容器

```bash
docker rename CONTAINER NEW_NAME

$ docker rename my_container my_new_container
```

## 获取容器日志

```bash
docker logs [OPTIONS] CONTAINER

# 可选参数
    --details：显示详细日志

    --follow, -f：跟踪日志输出

    --since：显示某个开始时间戳的所有日志

    --tail：仅列出最新N条容器日志，默认值all

    --timestamps, -t：显示时间戳

    --until：API 1.35+显示某个时间戳之前的日志（2013-01-02T13:23:37）或相对时间（42m for 42 minutes）
    
 
# 查看指定相对时间日志：
$ docker run --name test -d busybox sh -c "while true; do $(echo date); sleep 1; done"$ date
```

## 列出容器中运行的进程

```bash
$ docker top CONTAINER [ps OPTIONS]
```

## 容器内文件拷贝至主机

```bash
$ docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-

# 可选参数
-L :保持源目标中的链接

# 将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。
$ docker cp  96f7f14e99ab:/www /tmp/
```

# 其他操作

## 查看docker系统信息

```bash
docker info [OPTIONS]

# 可选参数
	--format , -f：格式化输出参数

# 格式化输出
$ docker info --format '{{json .}}'{"ID":"I54V:OLXT:HVMM:TPKO:JPHQ:CQCD:JNLC:O3BZ:4ZVJ:43XJ:PFHZ:6N2S","Containers":14, ...}
```

## 获取容器/镜像的元数据

```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]

# 可选参数
    --format , -f :指定返回值的模板文件。

    --size , -s :显示总的文件大小。

    --type :为指定类型返回JSON。
    
# 获取容器名为container_name的ip地址
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name

# 获取所有容器的Ip地址
$ docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)

# 获取容器的mac地址
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.MacAddress}}{{end}}' container_name

# 获取容器的日志地址
$ docker inspect --format='{{.LogPath}}' container_name

# 获取容器的镜像名称
$ docker inspect --format='{{.Config.Image}}' container_name

# 列出容器绑定的所有端口
$ docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' container_name
```
