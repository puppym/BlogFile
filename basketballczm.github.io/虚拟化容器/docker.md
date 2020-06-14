# docker 总的理解

docker 不是虚拟机，docker 容器的实质就是进程，但是直接在宿主执行的进程不同，容器进程运行于属于自己的独立命名空间，因此容器可以拥有自己的root文件系统，自己的额网络配置，自己的进程空间，甚至用户自己的用户ID空间。容器内的进程运行在一个隔离的环境里，使用起来就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主机中运行更加安全。正是这种隔离的特性，不要弄混淆容器和虚拟机。容器内的数据存储应该使用数据卷，任何存储于容器存储层的信息都会随容器的删除而丢失。
# docker命令

镜像加速器配置文件位置/etc/default/docker

* 启动docker 
systemctl enable docker
systemctl start docker 

systemctl stop docker  
* 一般只有root用户和docker用户组下的用户能够使用docker socket
* Dockerfile 定制镜像  Dockerfile的目录下 docker build -t nginx:v3   COPY 命令的目录是当前上下文目录
--no-cache 不是用缓存，每条指令都重新生成镜像，速度会很慢
-f 明确指明Dockerfile
-t 给生成的镜像打上标签。
Dockerfile文件中要有标准的输入输出的时候 RUN yum install -y XXX
比如输入的时候Enter yes Enter yes 
RUN sh -c '/bin/echo -e "\nyes\n\nyes" | sh Anaconda3-4.4.0-Linux-x86_64.sh'
从Dockerfile build docker image     docker build -t tag:version .
docker build - < Dockerfile 表示Dockerfile的内容从标准输入中获取

* docker run ubuntu:14.04 /bin/echo 'hello world'
启动一个bash终端，允许用户进行交互 docker run -t -i ubuntu:14.04 /bin/bash
-t 让Docker分配一个伪终端并绑定一个伪终端 
-i 则让容器的标准输入保持打开
-d 容器会自动进入后台

* docker container start 直接讲一个已经终止的容器进行启动运行。
* docker container ls  列出正在运行的容器
* docker contianer ls -a 列出已经停止的容器
* docker container logs [container ID or name] 获取容器的输出信息
* docker container stop 来终止一个运行的程序，容器中指定程序终止时，容器也就自动终止.
* docker commit containerID learn/ping
* exit 或者 ctrl+d 命令来退出终端
* docker contaihttp://example.com/exampleimage.tgz example/imagereponer ls -a 显示终止状态的容器 docker container restart 终止的容器重新启动
* docker attach    docker exec进入容器的操作，推荐用docker exec
docker attach [container id] exit 会导致容器停止
docker exec -i -t [contianer id] bash  -i 由于没有分配伪终端，界面没有我们熟悉的Linux命令提示符

* docker export [container id] > filename.tar 导出本地的某个容器
* cat ubuntu.tar | docker import - test/ubuntu:v1.0 从容器快照文件中再导入为镜像
* docker import  http://example.com/exampleimage.tgz example/imagerepo
* docker import将快照信息导入为镜像，这样将丢弃容器快照的所有历史记录和元数据信息。docker load 导入镜像存储文件到本地镜像库，奖项存储文件将保存完整的记录，体积也比较大，可以重新制定标签等元数据信息。
* docker container rm [contianer id ] 删除一个容器
* docker contianer prune 删除所有处于终止状态的容器
* docker search [imagesname]
* docker tag ubuntu:17.0 username/ubuntu:17.10
docker image ls
docker push username/ubuntu:17.10
docker search username

* 自动创建对于需要经常升级的镜像内程序来说十分方便，自动创建允许用户通过Docker Hub制定跟踪一个目标网站(目前支持github或BitBucket)，一旦项目发生新得提交或者创建新的标签(tag)，Docker Hub会自动构建镜像并且推送到Docker Hub中
* 通过 docker-registry 官方提供的工具来构建私有仓库。
docker run -d -p 5000:5000 --restart=always --name registery register 默认情况下仓库会被创建在容器的/var/lib/registery目录下，可以指定目录。后面可以在私有仓库上上传，搜索，下载镜像。后面进阶的学习中可以使用Docker Compose搭建一个拥有权限认证，TLS的私有仓库。
# 数据管理
* 数据卷是一个可供一个或多个容器使用的特殊目录，修改会立马生效，更新不影响镜像，数据卷一直存在，即使容器被删除
* docker volume create my-vol   -v 和 -mount 参数，建议使用-mount参数
docker volume ls
docker volume inspect my-vol 查看数据卷的信息
docker run -d -P --name web --mount source=my-vol,target=/webapp training/webapp python app.py 创建一个名为web的容器，并且加载一个数据卷到容器的/webapp目录下 -P 和 -p参数来指定端口映射，-P标记会随机映射一个49000到49900的端口，-p 是特定指定的hostPort:containerPort 5000端口映射到容器的5000端口
docker inspect web 检查容器的信息“Mounts”
docker volume rm my-vol
docker volume prune 清楚多余无主的数据卷
# 网络
* 访问外部网络 docker run -d -P training/webapp python app.py 
docker container ls -l
docker run -d -p 5000:5000 training/webapp python app.py
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py  ip:hostPort:containerPort
docker run -d -p 127.0.0.1::5000 training/webapp python app.py      ip ::containerPort绑定localhost的任意端口到容器的5000端口，本机自动分配端口
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
docker port 查看端口映射
docker run -d -p 5000:5000 -p 3000:80 trianing/webapp python app.py  多个端口的映射
* 容器互联 --link参数能够用来使得容器互联，随着Docker网络的完善，建议将容器加入自定义的Docker网络来连接多个容器。
docker network create -d bridge my-net 创建一个新网络 -d 指定网络类型 bridge和overlay
docker run -it --rm --name busybox1 --network my-net busybox sh  
docker run -it --rm --name busybox2 --network my-net busybox sh 
busybox1: ping busybox2
busybox2: ping busybox1
Docker compose 能够实现多个容器相互连接
# 配置DNS
* Docker利用虚拟文件来挂载容器的3个相关配置文件
mount 查看相关挂载信息1. mount，所有Docker容器的DNS配置通过/etc/resolv.conf文件立即得到更新 2. 修改、etc/docker、daemon.json 3.手动指定容器的配置，使用docker run命令加参数 -h   默认使用主机上的/etc/resolv.conf来配置容器
# docker 容器文件挂载
docker run -it -v /usr:/opt --name=my_second_container --ulimit='stack=-1:-1' klee/klee
gcc -I /opt/include /opt/local/
PATH=$PATH:/opt/local/sbin/:/opt/local/bin:/opt/sbin:/opt/bin

# 高级网络配置

##  快速配置指南

 * 有些命令只有在Docker服务启动的时候才能配置，而且不能马上生效
-b BRIDGE 或 --bridge=BRIDGE 指定容器挂载的网桥
--bip=CIDR                  定制 docker0 的掩码
-H SOCKET... 或 --host=SOCKET... Docker 服务端接收命令的通道
--icc=true|false 是否支持容器之间进行通信  (可以在 /etc/default/docker 文件中配置 DOCKER_OPTS=--icc=false 来禁止它。)
在通过 -icc=false 关闭网络访问后，还可以通过 --link=CONTAINER_NAME:ALIAS 选项来访问容器的开放端口(访问指定端口)
--ip-forward=true|false 请看下文容器之间的通信
--iptables=true|false 是否允许 Docker 添加 iptables 规则
--mtu=BYTES 容器网络中的 MTU
* 下面既可以在启动服务时指定，也可以在启动容器时指定。Docker 服务启动是指定后会覆盖默认值
--dns=IP_ADDRESS... 使用指定的DNS服务器
--dns-search=DOMAIN... 指定DNS搜索域
* 有些配置只有在docker run 执行时使用，因为他是针对容器的特征内容
-h HOSTNAME 或 --hostname=HOSTNAME 配置容器主机名
--link=CONTAINER_NAME:ALIAS 添加到另一个容器的连接
--net=bridge|none|container:NAME_or_ID|host 配置容器的桥接模式
-p SPEC 或 --publish=SPEC 映射容器端口到宿主主机
-P or --publish-all=true|false 映射容器所有端口到宿主主机
## 容器访问控制主要是通过linux上自带的iptables防火墙来进行管理和实现

 * 若访问外网 sysctl net.ipv4.ip_forward 
sysctl -w net.ipv4.ip_forward=1    启动docker服务时设定  --ip-forward=true Docker就会自动设定系统的ip_forward参数为1
* 容器间的访问，要两个方面的支持:
容器的网络拓扑是否已经互联。默认情况下，所有容器都会被连接到 docker0 网桥上。
本地系统的防火墙软件 -- iptables 是否允许通过。
## Docker0网桥

 服务器会默认创建一个Docker0 网桥，其上有一个docker0内部接口 它在内核层联通了其他的物理或虚拟网卡，这时将所有容器和本地主机都放到同一个物理网络。
 每次创建一个新容器的时候，Docker从可用的地址段中选择一个空闲的ip地址分配给容器的eth0端口。使用本机地址上docker0接口的IP作为所有容器的默认网关。
 在自定义网桥后要修改 /etc/docker/daemon.json 中的内容，将增加的网桥添加到配置文件中，即可将Docker默认桥接到创建的网桥上。

 * 创建一个点到点的连接