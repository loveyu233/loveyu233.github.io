---
title: "Docker"
date: 2021-01-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- docker
---

# Docker 

## 安装

```shell
下载需要的安装包
sudo yum install -y yum-utils
配置镜像仓库阿里云的
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
更新yum软件包索引
yum makecache fast
安装docker，docker-ce社区版
sudo yum install -y docker-ce docker-ce-cli containerd.io
启动docker
sudo systemctl start docker
查看docker
docker version
测试helloworld
sudo docker run hello-wo
```

## 卸载

```shell
卸载依赖
sudo yum remove docker-ce docker-ce-cli containerd.io
删除资源
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

## docker执行run之后做了哪些事情？

开始--->docker会在本机寻找镜像--->如果有就使用/没有就去dockerHub上下载--->dockerHub是否可以找到--->找不到返回错误/找到就下载镜像到本地

找不到报错：

[root@iZbp16y4pny75sbune791jZ /]# docker run hzyy
Unable to find image 'hzyy:latest' locally
docker: Error response from daemon: pull access denied for hzyy, repository does not exist or may require 'docker login': denied: requested access to the resource is denied.
See 'docker run --help'.

## 底层原理

### docker是怎么工作的？

docker是一个client-server结构的系统，docker的守护进程运行在主机上，通过socket从客户端访问！

dockerserver接收到docker-client的指令，然后运行该指令

### docker为什么比vm块？

docker的抽象层更加的少。

## docker命令

docker帮助命令

```shell
docker version		#docker的版本信息
docker info		#显示docker的系统信息
docker 命令 --help		#帮助命令
#docker官方帮助文档		https://docs.docker.com/reference/
```

## 镜像命令

```shell
搜索本地镜像
[root@iZwz9hjf71wfpyz4pppqreZ /]# docker images	
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    d1165f221234   3 weeks ago   13.3kB
#解释：
REPOSITORY	镜像的仓库源/名字
TAG					镜像的标签
IMAGE ID		镜像的id
CREATED			镜像的创建时间
SIZE				镜像的大小
#可选项
  -a, --all             显示全部的镜像
  -q, --quiet           只显示镜像的id


#搜索远程仓库镜像
[root@iZwz9hjf71wfpyz4pppqreZ /]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10680     [OK] 



#下载镜像
[root@iZwz9hjf71wfpyz4pppqreZ /]# docker pull mysql
Using default tag: latest	#如果不写tag默认就是最新版
latest: Pulling from library/mysql
a076a628af6f: Pull complete 	#分层下载，docker image的核心 联合文件系统
f6c208f3f991: Pull complete 
88a9455a9165: Pull complete 
406c9b8427c6: Pull complete 
7c88599c0b25: Pull complete 
25b5c6debdaf: Pull complete 
43a5816f1617: Pull complete 
1a8c919e89bf: Pull complete 
9f3cf4bd1a07: Pull complete 
80539cea118d: Pull complete 
201b3cad54ce: Pull complete 
944ba37e1c06: Pull complete 
Digest: sha256:feada149cb8ff54eade1336da7c1d080c4a1c7ed82b5e320efb5beebed85ae8c	#签名，防伪
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest	#真实地址

docker pull mysql:5.7	#指定版本下载

[root@iZwz9hjf71wfpyz4pppqreZ /]# docker rmi -f 容器id	删除指定容器id的镜像，删除多个镜像用空格隔开
[root@iZwz9hjf71wfpyz4pppqreZ /]# docker rmi -f $(docker images -aq)	删除全部镜像

docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hzy1214.. mysql
mysql运行时指定密码
-e MYSQL_ROOT_PASSWORD=123456
```

## 容器命令

有了镜像才可以创建容器

```shell
docker pull centos		#下载镜像
docker run [参数] image		#运行
--name = "Name" 容器名字
-d		后台方式运行
-it		交互方式运行，进入容器查看内容
-p小写的p		指定容器的端口
	-p ip：主机端口：容器端口
-P大写的P		随机指定端口

docker run -it centos /bin/bash #启动并进入容器
    [root@iZbp16y4pny75sbune791jZ /]# docker run -it centos /bin/bash
    [root@e8707443b8d9 /]# 
    
exit. #退出容器

[root@iZbp16y4pny75sbune791jZ /]# docker ps		#查看当前正在运行的容器
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@iZbp16y4pny75sbune791jZ /]# docker ps -a			#查看当前正在运行的容器，和历史运行过的容器
																						-n=1			#列出指定条容器
																						-q。     #只显示容器的编号
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                          PORTS     NAMES
e8707443b8d9   centos         "/bin/bash"   3 minutes ago    Exited (0) About a minute ago             dazzling_engelbart
11fb25f27465   d1165f221234   "/hello"      47 minutes ago   Exited (0) 47 minutes ago                 pensive_kirch
[root@iZbp16y4pny75sbune791jZ /]# 

```

## 启动和停止容器

```shell
docker start 容器id				启动容器
docker restart 容器id			重启容器
docker stop	容器id				停止运行的容器
docker kill 容器id 				强制停止容器
```

## 常用其他命令

```shell
docker run -d centos 			#后台启动容器
	docker ps发现容器停止了
	docker容器使用后台运行，就必须要有一个前台进程，docker发现没有前台应用就会自动停止
```

### 查看日志

```shell
docker logs -f -t 容器			查看容器全部日志
docker logs -tf --tail num  容器			查看num条数的日志
```

### 查看容器中进程信息

```shell
docker top 容器id
UID：用户id
PID：进程id
PPID：父进程id
```

### 查看镜像元数据

```shell
docker inspect 容器id
```

### 进入当前正在运行的容器

```shell
docker exec -it 容器id /bin/bash	进入容器后开启一个新的终端
docker attach 容器id			进入容器正在执行的终端，不会启动新的进程
```

### 从容器中拷贝内容到主机中

```shell
docker copy 容器id：容器内路径 主机路径
```

## 练习

### 部署nginx

```shell
docker search nginx		#搜索镜像
docker pull nginx		#下载镜像
docker images			#查看
docker run -d --name nginx01 -p 3344:80 nginx			#-d后台运行--name起一个别名nginx01-p暴露端口相当于主机的3344端口关联到容器nginx的80端口，外网访问3344端口到容器nginx
docker ps  #查看进程
[root@iZbp16y4pny75sbune791jZ /]# curl localhost:3344		#测试端口
[root@iZbp16y4pny75sbune791jZ /]# docker exec -it nginx01 /bin/bash		#交互式进入nginx
root@12740e8ba552:/# whereis nginx		#查找nginx文件位置
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@12740e8ba552:/# cd /etc/nginx		#进入文件位置
docker stop 12		#停止容器 12是容器id的缩写
```

### 部署tomcat

```shell
docker run -it --rm tomcat:9.0				#rm用来测试的用完就自动删除容器，但是镜像还在

docker run -d -p 3344:8080 --name tom tomcat   #后台启动

docker exec -it tom /bin/bash   #进入容器

发现问题：	linux命令少了，没有webapps。因为阿里云镜像默认是最小的，所有不必要的都剔除了，保证最小可运行环境

webapps里面的少的文件在webapps.dist里面
cp -r webapps.dist/* webapps  #拷贝到webapps里面
```

### 部署es+kibana

```shell
#es 暴露的端口很多
#es 十分的耗内存
#es 的数据一般需要放置到安全目录！要用到挂载

docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

docker stats 查看cpu状态


-e ES_JAVA_OPTS="-Xms64m -Xmx512m" #限制es使用的内存
docker run -d --name elasticsearch2 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2


```

## 可视化面板

```shell
docker run -d -p 8088:9000 \
> --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

## 提交自己的镜像

### commit镜像

```
docker commit 提交容器成为一个新的副本
docker commit -m=“提交的描述信息” -a=“作者” 容器id 目标镜像名 [TAG]
docker commit -a="hzyy" -m="add webapps" 828 tom:1.0
```

### 过程

>第一步：pull原版的tomcat
>
>发现原版的webapps里面没有文件
>
>第二步：运行打开原版tomcat，添加文件到webapps里面，然后退出
>
>第三步：docker commit -a="hzyy" -m="add webapps" 828 tom:1.0
>
> -a 作者 -m 描述 828 镜像id的简写 tom:1.0 镜像名称和版本
>
>提交了修改后的镜像到本地镜像库，然后使用这个镜像里面的webapps里面就有文件了

### 理解

>docker的分层原理

## 容器数据卷

如果数据都在容器中，那么我们容器删除，数据就会丢失， 需求：数据可以持久化 

容器之间可以有一个数据共享的技术，docker容器中产生的数据，同步到本地！

这就是卷技术，目录的挂载，将我们容器内的目录，挂载到Linux上面！

 总结 ：容器的持久化和同步操作！容器间也是可以数据共享的

### 方式一：直接使用命令来挂载

docker run -it -v 主机目录：容器目录

```shell
docker run -it -v /home/ceshi:/home centos /bin/bash
									主机的文件夹  容器的文件夹
									双向的
									
									/home/nacos/conf
```

### 部署mysql

思考：mysql的数据持久化

>docker pull mysql		
>
>docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=hzyy1214.. --name mysql
>
>-d 后台运行
>
>-p 端口映射
>
>-v 卷挂载
>
>--name 容器别名

### 具名和匿名挂载

>### 匿名挂载：
>
>不指定本机的文件夹
>
>查看卷情况
>
>docker volume ls
>
>
>
>### 具名挂载：
>
>卷名：容器内路径，不指定本机的具体路径
>
>查看容器挂载本机卷位置
>
>docker volume inspect 名称
>
>这个名称就是用docker volume ls查看出来的
>
>
>
>### 本机路径/文件夹：容器路径/文件夹	这是置顶路径挂载

#### 扩展：本机路径/文件夹：ro/rw 只读/读写



### 方式二：dockerfile

>创建dockerfile文件，名字随便
>
>文件内容
>
>FROM centos
>
>VOLUME ["volume01","volume02"]
>
>CMD echo "----end----"
>CMD /bin/bash
>
>这里的每一个命令就是镜像的一层
>
>```shell
>[root@iZbp16y4pny75sbune791jZ test]# docker run -it 79
>[root@43f1c304d566 /]# ls -ll
>total 56
>lrwxrwxrwx  1 root root    7 Nov  3 15:22 bin -> usr/bin
>drwxr-xr-x  5 root root  360 May  2 02:29 dev
>drwxr-xr-x  1 root root 4096 May  2 02:29 etc
>drwxr-xr-x  2 root root 4096 Nov  3 15:22 home
>lrwxrwxrwx  1 root root    7 Nov  3 15:22 lib -> usr/lib
>lrwxrwxrwx  1 root root    9 Nov  3 15:22 lib64 -> usr/lib64
>drwx------  2 root root 4096 Dec  4 17:37 lost+found
>drwxr-xr-x  2 root root 4096 Nov  3 15:22 media
>drwxr-xr-x  2 root root 4096 Nov  3 15:22 mnt
>drwxr-xr-x  2 root root 4096 Nov  3 15:22 opt
>dr-xr-xr-x 89 root root    0 May  2 02:29 proc
>dr-xr-x---  2 root root 4096 Dec  4 17:37 root
>drwxr-xr-x 11 root root 4096 Dec  4 17:37 run
>lrwxrwxrwx  1 root root    8 Nov  3 15:22 sbin -> usr/sbin
>drwxr-xr-x  2 root root 4096 Nov  3 15:22 srv
>dr-xr-xr-x 13 root root    0 May  1 09:24 sys
>drwxrwxrwt  7 root root 4096 Dec  4 17:37 tmp
>drwxr-xr-x 12 root root 4096 Dec  4 17:37 usr
>drwxr-xr-x 20 root root 4096 Dec  4 17:37 var
>drwxr-xr-x  2 root root 4096 May  2 02:29 volume01		#这就是挂载的卷
>drwxr-xr-x  2 root root 4096 May  2 02:29 volume02
>```
>
>docker inspect 52  查看卷挂载的本机路径
>
>```shell
>"Mounts": [
>   {
>       "Type": "volume",
>       "Name": "e74abd83256b630e334fac24771f989488131db858c00fc65c4e2e794fca6bd5",
>       "Source": "/var/lib/docker/volumes/e74abd83256b630e334fac24771f989488131db858c00fc65c4e2e794fca6bd5/_data", #位置1
>       "Destination": "volume01",
>       "Driver": "local",
>       "Mode": "",
>       "RW": true,
>       "Propagation": ""
>   },
>   {
>       "Type": "volume",
>       "Name": "a30e317f59d892c1bbd2a47784892c7323d897dee08b60a8a0a9a3f7d65c90bc",
>       "Source": "/var/lib/docker/volumes/a30e317f59d892c1bbd2a47784892c7323d897dee08b60a8a0a9a3f7d65c90bc/_data", #位置2
>       "Destination": "volume02",
>       "Driver": "local",
>       "Mode": "",
>       "RW": true,
>       "Propagation": ""
>   }
>],
>```

## 数据卷容器

容器之间挂载卷

>```
>[root@iZbp16y4pny75sbune791jZ _data]# docker run -it --name docker01 79
>[root@830126d40722 /]# ls
>bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var       volume02
>dev  home  lib64  media       opt  root  sbin  sys  usr  volume01
>[root@830126d40722 /]# 
>```
>
>docker run -it --name docker02 --volumes-from docker01 79		创建docker02挂载容器docker01

## dockerFile

核心是用来构建dockr镜像的文件，就是命令参数脚本

构建步骤：

- 编写dockerfile文件

- docker build 构建成为一个镜像
- docker run 运行镜像
- docker push 发布镜像（dockerhub或者阿里云镜像）

基础知识：

- 每个保留关键字（指令）都必须是大写字母
- 执行从上到下顺序执行
- #表示注释
- 每一个指令都会创建提交一个新的镜像层，并提交

## 保留字

FROM

>基础镜像，当前新镜像是基于那个镜像的 

MAINTAINER

>镜像维护者的姓名和邮箱地址

RUN

>容器构建时需要运行的命令

EXPOSE

>当前容器对外暴露的端口号

WORKDIR

>指定在创建容器后，终端默认登录进来的工作目录

ENV

>设置当前镜像的环境变量

ADD

>将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包

COPY

>将宿主机目录下的文件拷贝进镜像但不会自动解压

VOLUME

>容器数据卷，用于数据保存和持久化工作

CMD

>指定一个容器启动时要运行的命令
>
>dokerFIle可以有多个cmd命令但只有最后一个生效
>
>cmd会被docker run之后的参数替换

ENTRYPOINT

>指定一个容器启动时要运行的命令
>
>和CMD一样，都是在指定容器启动程序及参数
>
>ENTRYPOINT会被docker run之后的参数追加

ONBUILD

>当构建一个被继承的dockerFile时运行命令，父镜像在被子镜像继承后父镜像的ONBUILD被触发



>自定义dockerFile

```
FROM centos
ENV myPath /tem
WORKDIR $myPath
MAINTAINER hzyy<195@qq.com>
RUN yum -y install vim
CMD echo $myPath
CMD echo "success....."
```

>docker build -f dockerFile1 -t mytest:1.2 .

```shell
[root@iZbp16y4pny75sbune791jZ dockerFile]# docker build -f dockerFile1 -t mytest:1.2 .
Sending build context to Docker daemon  2.048kB
Step 1/7 : FROM centos
...
Successfully built f4024962b93e
Successfully tagged mytest:1.2
```



>docker run -it f4024962b93e /bin/bash

```
[root@iZbp16y4pny75sbune791jZ dockerFile]# docker run -it f4024962b93e /bin/bash
[root@36be08b76e60 tem]# pwd
/tem
```

#### 实战测试：

>## 创建一个自己的centos
>
>### 第一步创建脚本文件名称为myDockerFile：
>
>```shell
>[root@iZbp16y4pny75sbune791jZ hzyy]# vim myDockerFile
>[root@iZbp16y4pny75sbune791jZ hzyy]# ls
>myDockerFile  test
>[root@iZbp16y4pny75sbune791jZ hzyy]# cat myDockerFile 
>
>#脚本内容：
>FROM centos		#基础镜像
>MAINTAINER hzyy<195@qq.com>			#这个镜像的作者
>
>ENV MYPATH /usr/local			#进入这个镜像后的目录，默认进去是/路径设置后就会到指定的路径
>WORKDIR $MYPATH				#环境变量
>
>RUN yum -y install vim			#下载一个vim指令
>RUN yum -y install net-tools			#同理
>
>EXPOSE 80				#暴露端口
>
>CMD echo $MYPATH			#echo打印
>CMD echo "----end----"
>CMD /bin/bash				#命令台
>```
>
>第二步build这个脚本文件
>
>```
>[root@iZbp16y4pny75sbune791jZ hzyy]# docker build -f myDockerFile -t mycentos:2.0 .
>```
>
>第三步查看docker镜像
>
>```
>docker images
>```
>
>第四步运行这个镜像
>
>```
>[root@iZbp16y4pny75sbune791jZ hzyy]# docker run -it 31
>[root@c3ecfb3c3fde local]# ls
>bin  etc  games  include  lib  lib64  libexec  sbin  share  src
>[root@c3ecfb3c3fde local]# pwd
>/usr/local
>[root@c3ecfb3c3fde local]# 
>```
>
>### docker history 31 : 查看镜像的构建过程

### CMD和ENTRYPOINT区别

cmd：指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代

entrypoint：指定这个容器启动的时候要运行的命令，可以追加命令



### 实战：tomcat镜像



### 镜像发布到dockerhub

>docker login -u hzyy			登录账号
>
>[root@iZbp16y4pny75sbune791jZ hzyy]# docker push hzyy/cmdtest			发布，报错原因没有标签
>Using default tag: latest
>The push refers to repository [docker.io/hzyy/cmdtest]
>An image does not exist locally with the tag: hzyy/cmdtest
>
>解决：
>
>docker tag cmdtest hzyy/test:1.0			docker tag 镜像名称 标签：版本号
>
>[root@iZbp16y4pny75sbune791jZ hzyy]# docker push hzyy/test:1.0			发布成功
>The push refers to repository [docker.io/hzyy/test]
>2653d992f4ef: Mounted from library/centos 
>1.0: digest: sha256:74a3aabad5ec815f6d010cbcd2ef868fb27553c388ea1db66e253a857f942b3e size: 529

### 镜像发布到阿里云

https://cr.console.aliyun.com/repository/cn-hangzhou/h_xiaoyu/xiaoyu/details

## docker网路

>[root@iZbp16y4pny75sbune791jZ ~]# ip addr
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>inet 127.0.0.1/8 scope host lo
>  valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>link/ether 00:16:3e:18:47:30 brd ff:ff:ff:ff:ff:ff
>inet 172.19.29.40/18 brd 172.19.63.255 scope global dynamic eth0
>  valid_lft 315274096sec preferred_lft 315274096sec
>3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
>link/ether 02:42:71:31:2b:5f brd ff:ff:ff:ff:ff:ff
>inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
>  valid_lft forever preferred_lft forever
>
>1.本机回环地址
>
>2.阿里云内网地址
>
>3.dockero 地址

>查看容器网络地址
>
>[root@iZbp16y4pny75sbune791jZ ~]# docker exec -it 8a ip addr
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>inet 127.0.0.1/8 scope host lo
>  valid_lft forever preferred_lft forever
>52: eth0@if53: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
>link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
>inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
>  valid_lft forever preferred_lft forever
>
>### 本机ping容器
>
>[root@iZbp16y4pny75sbune791jZ ~]# ping 172.17.0.2
>PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
>64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.081 ms
>64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms
>64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.085 ms
>64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.063 ms
>64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.061 ms
>64 bytes from 172.17.0.2: icmp_seq=6 ttl=64 time=0.065 ms
>64 bytes from 172.17.0.2: icmp_seq=7 ttl=64 time=0.076 ms
>
>### 原理：
>
>每启动一个docker容器，docker就会给docker容器分配一个ip，只要安装了docker，就会有一个网卡docker0
>
>桥接模式，使用的激素是veth-pair技术
>
>再次测试ip addr
>
>[root@iZbp16y4pny75sbune791jZ ~]# ip addr
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>inet 127.0.0.1/8 scope host lo
>  valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>link/ether 00:16:3e:18:47:30 brd ff:ff:ff:ff:ff:ff
>inet 172.19.29.40/18 brd 172.19.63.255 scope global dynamic eth0
>  valid_lft 315273494sec preferred_lft 315273494sec
>3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
>link/ether 02:42:71:31:2b:5f brd ff:ff:ff:ff:ff:ff
>inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
>  valid_lft forever preferred_lft forever
>53:  vethd 4f86d2@if52: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
>link/ether 16:e5:11:d0:45:d3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
>
>#### 发现多了53和52
>
>### 当再打开一个容器发现又多了一对
>
>发现这个容器带来网卡都是一对对的
>
>veth-pair 就是一对的虚拟设备接口，它们都是成对出现的，一段连着协议，一段彼此相连
>
>正因为有这个特性，veth-pair 充当一个桥梁，连接各种虚拟网络设备的

>结论：
>
>docker的容器和容器之间是可以互相ping通的

## --link

docker run -d -P --name tomcato3 --link tomcato2 tomcat

>
>
>[root@iZbp16y4pny75sbune791jZ ~]# docker exec -it e7 cat /etc/hosts
>127.0.0.1       localhost
>::1     localhost ip6-localhost ip6-loopback
>fe00::0 ip6-localnet
>ff00::0 ip6-mcastprefix
>ff02::1 ip6-allnodes
>ff02::2 ip6-allrouters
> 172.17.0.2      3a 3a19026df1bd tomcato2 
>172.17.0.3      e7f675f21dd6

现在不推荐使用link了，推荐使用自定义网路，因为docker0不支持容器名访问



## 自定义网路

[root@iZbp16y4pny75sbune791jZ ~]# docker network ls		查看docker网络



>[root@iZbp16y4pny75sbune791jZ ~]# docker network ls
>NETWORK ID     NAME      DRIVER    SCOPE
>77a47b86ebe2   bridge    bridge    local
>dae8b7a30e0a   host      host      local
>347c45f59dbe   none      null      local

### 网络模式

bridge：桥接 docker默认，自己创建网络也是这个

none：不配置网络

host：和宿主机共享网络

container：容器间网络连通（用得少）

测试

>直接启动的命令 --net bridge 这个就是docker0，不写默认就是的
>
>[root@iZbp16y4pny75sbune791jZ ~]# docker run -d -P --name tomcat01 --net bridge tomcat

```shell
#docker0特点：默认，容器名不能访问，--link可以打通连接

[root@iZbp16y4pny75sbune791jZ ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
76faec2782431ab1a0ac575231aa84b3aa415721b642cb260d00261cdc40ae11

#--driver bridge桥接模式
#--subnet 192.168.0.0/16 子网地址
#--gateway 192.168.0.1  网关


[root@iZbp16y4pny75sbune791jZ ~]# docker network ls				查看网络
NETWORK ID     NAME      DRIVER    SCOPE
77a47b86ebe2   bridge    bridge    local
dae8b7a30e0a   host      host      local
76faec278243   mynet     bridge    local
347c45f59dbe   none      null      local


[root@iZbp16y4pny75sbune791jZ ~]# docker network inspect mynet			查看详细信息
[
    {
        "Name": "mynet",
        "Id": "76faec2782431ab1a0ac575231aa84b3aa415721b642cb260d00261cdc40ae11",
        "Created": "2021-05-02T18:04:14.984438268+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

```shell
#创建并查看
[root@iZbp16y4pny75sbune791jZ ~]# docker run -d -P --name romcat01 --net mynet tomcat
d530c1e4f8edaf89ad69ab619eaba111f314e579d96433041a19c2e8a2dce63f
[root@iZbp16y4pny75sbune791jZ ~]# docker run -d -P --name romcat02 --net mynet tomcat
87ba86533bce073648fc28e62af8db9d93bee1da3886efa54577c2f21535cc67
[root@iZbp16y4pny75sbune791jZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED         STATUS         PORTS                                         NAMES
87ba86533bce   tomcat    "catalina.sh run"   3 seconds ago   Up 2 seconds   0.0.0.0:49157->8080/tcp, :::49157->8080/tcp   romcat02
d530c1e4f8ed   tomcat    "catalina.sh run"   9 seconds ago   Up 8 seconds   0.0.0.0:49156->8080/tcp, :::49156->8080/tcp   romcat01
[root@iZbp16y4pny75sbune791jZ ~]# docker inspect
"docker inspect" requires at least 1 argument.
See 'docker inspect --help'.

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects
[root@iZbp16y4pny75sbune791jZ ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "76faec2782431ab1a0ac575231aa84b3aa415721b642cb260d00261cdc40ae11",
        "Created": "2021-05-02T18:04:14.984438268+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "87ba86533bce073648fc28e62af8db9d93bee1da3886efa54577c2f21535cc67": {
                "Name": "romcat02",
                "EndpointID": "3a6b863971e2bee65e411125e943c4e413f5eeff30321bcb3dddaea08ef23f0c",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "d530c1e4f8edaf89ad69ab619eaba111f314e579d96433041a19c2e8a2dce63f": {
                "Name": "romcat01",
                "EndpointID": "36a5ee0cd2577af37de185bfe1c8e40fb9663ea3189d24d885c70940d7c43db2",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

```shell
[root@iZbp16y4pny75sbune791jZ ~]# docker exec -it 87 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.052 ms
```

## 网络连通

docker network connect mynet romcat-d-01

​										mynet：自定义的网络	romcat-d-01要加入该网络的容器

加入之后再同网络下就可以ping通了



## 实战redis集群

### 第一步：创建网络

>[root@iZbp16y4pny75sbune791jZ ~]#  docker network create redis --subnet 172.38.0.0/16 
>4a375688fa00ee4122dadf3fd4aeedc2e6fda94c7222d7071ab4a8157c3bb5bb
>
>[root@iZbp16y4pny75sbune791jZ ~]# docker network ls 
>NETWORK ID     NAME      DRIVER    SCOPE
>77a47b86ebe2   bridge    bridge    local
>dae8b7a30e0a   host      host      local
>76faec278243   mynet     bridge    local
>347c45f59dbe   none      null      local
>4a375688fa00   redis     bridge    local
>
>[root@iZbp16y4pny75sbune791jZ ~]#  docker inspect redis 
>[
>{
>   "Name": "redis",
>   "Id": "4a375688fa00ee4122dadf3fd4aeedc2e6fda94c7222d7071ab4a8157c3bb5bb",
>   "Created": "2021-05-02T20:56:27.978799535+08:00",
>   "Scope": "local",
>   "Driver": "bridge",
>   "EnableIPv6": false,
>   "IPAM": {
>       "Driver": "default",
>       "Options": {},
>       "Config": [
>           {
>                "Subnet": "172.38.0.0/16" 
>           }
>       ]
>   },
>   "Internal": false,
>   "Attachable": false,
>   "Ingress": false,
>   "ConfigFrom": {
>       "Network": ""
>   },
>   "ConfigOnly": false,
>   "Containers": {},
>   "Options": {},
>   "Labels": {}
>}
>]

### 第二步：写一个脚本一次性创建六个redis配置

>for port in $(seq 1 6);
>
>do
>
>mkdir -p /mydate/redis/node-${port}/conf
>
>touch /mydata/redis/node-${port}/conf/redis.conf
>
>cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
>
>port 6379
>
>bind 0.0.0.0
>
>cluster-enabled yes
>
>cluster-config-file nodes.conf
>
>cluster-node-timeout 5000
>
>cluster-announce-ip 172.38.0.1${port}
>
>cluster-announce-port 6379
>
>cluster-announce-bus-port 16379
>
>appendonly yes
>
>EOF
>
>done

### 第三步：运行镜像

>docker run -p 6379:6379 -p16379:16379 --name redis-1
>
>-v /mydata/redis/node-1/data:/data
>
>-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf
>
>-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
>
>变为一段：
>
>docker run -p 6379:6379 -p16379:16379 --name redis-1 -v /mydata/redis/node-1/data:/data -v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

>```shell
>[root@iZbp16y4pny75sbune791jZ conf]# docker run -p 6379:6379 -p16379:16379 --name redis-1 \
>> -v /mydata/redis/node-1/data:/data \
>> -v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
>> -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
>```

### 第四步：构建集群

>```shell
>/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:
>6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>>> Performing hash slots allocation on 6 nodes...
>Master[0] -> Slots 0 - 5460
>Master[1] -> Slots 5461 - 10922
>Master[2] -> Slots 10923 - 16383
>Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
>Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
>Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
>M: 9fbfa50f8126d9f001640d1d6236cb75e2147d8c 172.38.0.11:6379
>slots:[0-5460] (5461 slots) master
>M: 902d0715973cd17de0122f350772fbc539830471 172.38.0.12:6379
>slots:[5461-10922] (5462 slots) master
>M: 6ed9b6c7080fabc37e865ad8d408745a7ffac986 172.38.0.13:6379
>slots:[10923-16383] (5461 slots) master
>S: f4b68b512ad48635df13434c095fb0b0493abe16 172.38.0.14:6379
>replicates 6ed9b6c7080fabc37e865ad8d408745a7ffac986
>S: 8c273888e82f9b6311ad36b5f9ac4c49ae841d7e 172.38.0.15:6379
>replicates 9fbfa50f8126d9f001640d1d6236cb75e2147d8c
>S: 7f0db03061806e7316bb21f248dea023f63de6b9 172.38.0.16:6379
>replicates 902d0715973cd17de0122f350772fbc539830471
>Can I set the above configuration? (type 'yes' to accept): yes
>>>> Nodes configuration updated
>>>> Assign a different config epoch to each node
>>>> Sending CLUSTER MEET messages to join the cluster
>Waiting for the cluster to join
>..
>>>> Performing Cluster Check (using node 172.38.0.11:6379)
>M: 9fbfa50f8126d9f001640d1d6236cb75e2147d8c 172.38.0.11:6379
>slots:[0-5460] (5461 slots) master
>1 additional replica(s)
>S: f4b68b512ad48635df13434c095fb0b0493abe16 172.38.0.14:6379
>slots: (0 slots) slave
>replicates 6ed9b6c7080fabc37e865ad8d408745a7ffac986
>S: 7f0db03061806e7316bb21f248dea023f63de6b9 172.38.0.16:6379
>slots: (0 slots) slave
>replicates 902d0715973cd17de0122f350772fbc539830471
>S: 8c273888e82f9b6311ad36b5f9ac4c49ae841d7e 172.38.0.15:6379
>slots: (0 slots) slave
>replicates 9fbfa50f8126d9f001640d1d6236cb75e2147d8c
>M: 902d0715973cd17de0122f350772fbc539830471 172.38.0.12:6379
>slots:[5461-10922] (5462 slots) master
>1 additional replica(s)
>M: 6ed9b6c7080fabc37e865ad8d408745a7ffac986 172.38.0.13:6379
>slots:[10923-16383] (5461 slots) master
>1 additional replica(s)
>[OK] All nodes agree about slots configuration.
>>>> Check for open slots...
>>>> Check slots coverage...
>[OK] All 16384 slots covered.
>/data # 
>```
>
>### 进入查看
>
>```shell
>/data # redis-cli -c
>127.0.0.1:6379> cluster info
>cluster_state:ok
>cluster_slots_assigned:16384
>cluster_slots_ok:16384
>cluster_slots_pfail:0
>cluster_slots_fail:0
>cluster_known_nodes:6
>cluster_size:3
>cluster_current_epoch:6
>cluster_my_epoch:1
>cluster_stats_messages_ping_sent:310
>cluster_stats_messages_pong_sent:314
>cluster_stats_messages_sent:624
>cluster_stats_messages_ping_received:309
>cluster_stats_messages_pong_received:310
>cluster_stats_messages_meet_received:5
>cluster_stats_messages_received:624
>127.0.0.1:6379> cluster nodes
>f4b68b512ad48635df13434c095fb0b0493abe16 172.38.0.14:6379@16379 slave 6ed9b6c7080fabc37e865ad8d408745a7ffac986 0 1619962726319 4 connected
>7f0db03061806e7316bb21f248dea023f63de6b9 172.38.0.16:6379@16379 slave 902d0715973cd17de0122f350772fbc539830471 0 1619962725015 6 connected
>9fbfa50f8126d9f001640d1d6236cb75e2147d8c 172.38.0.11:6379@16379 myself,master - 0 1619962724000 1 connected 0-5460
>8c273888e82f9b6311ad36b5f9ac4c49ae841d7e 172.38.0.15:6379@16379 slave 9fbfa50f8126d9f001640d1d6236cb75e2147d8c 0 1619962727321 5 connected
>902d0715973cd17de0122f350772fbc539830471 172.38.0.12:6379@16379 master - 0 1619962725517 2 connected 5461-10922
>6ed9b6c7080fabc37e865ad8d408745a7ffac986 172.38.0.13:6379@16379 master - 0 1619962726821 3 connected 10923-16383
>127.0.0.1:6379>  
>```
>
>

>## 测试
>
>```shell
>127.0.0.1:6379> set a b					#3号进行的操作
>docker stop redis-3							#停止3号
>/data # redis-cli -c						#重连
>127.0.0.1:6379> get a						#获取
>-> Redirected to slot [15495] located at 172.38.0.14:6379				#拿到3号从机4号上线作为mster
>"b"
>172.38.0.14:6379> cluster nodes
>f4b68b512ad48635df13434c095fb0b0493abe16 172.38.0.14:6379@16379 myself,master - 0 1619962994000 7 connected 10923-16383
>8c273888e82f9b6311ad36b5f9ac4c49ae841d7e 172.38.0.15:6379@16379 slave 9fbfa50f8126d9f001640d1d6236cb75e2147d8c 0 1619962995000 5 connected
>9fbfa50f8126d9f001640d1d6236cb75e2147d8c 172.38.0.11:6379@16379 master - 0 1619962995151 1 connected 0-5460
>902d0715973cd17de0122f350772fbc539830471 172.38.0.12:6379@16379 master - 0 1619962995050 2 connected 5461-10922
>7f0db03061806e7316bb21f248dea023f63de6b9 172.38.0.16:6379@16379 slave 902d0715973cd17de0122f350772fbc539830471 0 1619962995151 6 connected
>6ed9b6c7080fabc37e865ad8d408745a7ffac986 172.38.0.13:6379@16379 master,fail - 1619962822292 1619962821681 3 connected
>172.38.0.14:6379> 
>```
>
>

## 实战springboot打包docker镜像

1.构架springboot项目

2.打包应用

3.编写dockerfile

>```
>FROM java:8
>
>COPY *.jar demo-0.0.1-SNAPSHOT.jar
>
>EXPOSE 8080
>
>ENTRYPOINT ["java","-jar","demo-0.0.1-SNAPSHOT.jar"]
>```

4.构建镜像

>
>
>```shell
>[root@iZbp16y4pny75sbune791jZ idea]# docker build -t hzyy .
>[root@iZbp16y4pny75sbune791jZ idea]# docker images
>REPOSITORY   TAG                IMAGE ID       CREATED          SIZE
>hzyy         latest             707585e2437b   30 seconds ago   660MB
>tomcat       latest             c0e850d7b9bb   9 days ago       667MB
>redis        5.0.9-alpine3.11   3661c84ee9d0   12 months ago    29.8MB
>java         8                  d23bdf5b1b1b   4 years ago      643MB
>```
>
>

5.发布运行

>```
>[root@iZbp16y4pny75sbune791jZ idea]# docker run -d -p 8080:8080 --name hzyy hzyy
>801d877deac3844e1c4bd4546876bf11158622a322472843a537b52bbc1d64df
>[root@iZbp16y4pny75sbune791jZ idea]# docker ps
>CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
>801d877deac3   hzyy      "java -jar demo-0.0.…"   6 seconds ago   Up 6 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   hzyy
>[root@iZbp16y4pny75sbune791jZ idea]# curl localhost:8080
>{"timestamp":"2021-05-02T14:50:04.072+00:00","status":404,"error":"Not Found","message":"","path":"/"}[root@iZbp16y4pny75sbune791jZ idea]# ls
>D  demo-0.0.1-SNAPSHOT.jar  Dockerfile
>[root@iZbp16y4pny75sbune791jZ idea]# vim Dockerfile 
>```
>
>

# Docker Compose

docker compose用来定义和运行多个容器，用YAML file配置文件。

作用：批量容器编排

compose是docker官方的开源项目，需要安装

dockerfile让程序可以在任何地方运行,但是如果多个容器的话需要一个一个启动，麻烦！

compose的yaml文件，可以一次性运行多个容器。

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

服务services:容器，应用。（web，redis，mysql...）

项目project：一组关联的容器

## 安装

#### 1.下载

> sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 

#### 2.授权

> sudo chmod +x /usr/local/bin/docker-compose 

#### 3.查看版本确定安装成功

>[root@iZbp16y4pny75sbune791jZ bin]#  docker-compose version 
>docker-compose version 1.29.1, build c34c88b2
>docker-py version: 5.0.0
>CPython version: 3.7.10
>OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
>[root@iZbp16y4pny75sbune791jZ bin]# 

#### 4.快速体验

- 创建文件夹进入

  - >mkdir composetest
    >cd composetest

- 应用app.py

  - >创建app.py
    >
    >```python
    >import time
    >
    >import redis
    >from flask import Flask
    >
    >app = Flask(__name__)
    >cache = redis.Redis(host='redis', port=6379)
    >
    >def get_hit_count():
    >retries = 5
    >while True:
    >   try:
    >       return cache.incr('hits')
    >   except redis.exceptions.ConnectionError as exc:
    >       if retries   0:
    >           raise exc
    >       retries -= 1
    >       time.sleep(0.5)
    >
    >@app.route('/')
    >def hello():
    >count = get_hit_count()
    >return 'Hello World! I have been seen {} times.\n'.format(count)
    >```
    >
    >创建requirements.txt
    >
    >```
    >flask
    >redis
    >```

- dockerfile打包镜像

  - >创建dockerfile
    >
    >```
    ># syntax=docker/dockerfile:1
    >FROM python:3.7-alpine
    >WORKDIR /code
    >ENV FLASK_APP=app.py
    >ENV FLASK_RUN_HOST=0.0.0.0
    >RUN apk add --no-cache gcc musl-dev linux-headers
    >COPY requirements.txt requirements.txt
    >RUN pip install -r requirements.txt
    >EXPOSE 5000
    >COPY . .
    >CMD ["flask", "run"]
    >```

- docker-compose.yaml文件（定义整个服务需要的环境）

  - >创建docker-compose.yml
    >
    >```
    >version: "3.9"
    >services:
    >web:
    >build: .
    >ports:
    >     - "5000:5000"
    > redis:
    >   image: "redis:alpine"
    >```

- 启动compose项目

  - >docker-compose up

## 流程

1.创建网络

2.执行docker-compose.yaml

3.启动服务

# docker Swarm

集群方式的部署



https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/



节点：管理节点，工作节点

管理节点可以控制工作节点，但是工作节点不能控制管理节点

操作都在控制节点上

raft一致性算法

![截屏2021-05-03 下午2.34.16](../../../../../../Desktop/%25E6%2588%25AA%25E5%25B1%258F2021-05-03%2520%25E4%25B8%258B%25E5%258D%25882.34.16.png)

## 搭建集群

>[root@iZbp16y4pny75sbune791jZ hzyy]# docker swarm --help
>
>Usage:  docker swarm COMMAND
>
>Manage Swarm
>
>Commands:
>ca          Display and rotate the root CA
>init        Initialize a swarm
>join        Join a swarm as a node and/or manager
>join-token  Manage join tokens
>leave       Leave the swarm
>unlock      Unlock swarm
>unlock-key  Manage the unlock key
>update      Update the swarm
>
>Run 'docker swarm COMMAND --help' for more information on a command.



>
>
>[root@iZbp16y4pny75sbune791jZ hzyy]# docker swarm init --advertise-addr 172.19.29.40
>Swarm initialized: current node (r2whwa8mr5obzjh4m8no23mxf) is now a manager.
>
>To add a worker to this swarm, run the following command:
>
>docker swarm join --token SWMTKN-1-5a0mwmc3157mjoywht2isz2qw98vg8xzuld140x1jfndz6u7zd-cmh58tk0bl5qo08kz506mfbjk 172.19.29.40:2377
>
>To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
>
>[root@iZbp16y4pny75sbune791jZ hzyy]# 

初始化节点： docker swarm init --advertise-addr 120.26.11.225

加入一个节点：docker swarm join

>获取令牌
>
>docker swarm join-token manager			管理节点
>
>docker swarm join-token worker				工作节点

systemctl top docker		下线

systemctl start docker		上线



docker node promote swarm4升级

docker node demote swarm4降级



>  rollback    Revert changes to a service's configuration		回滚
>  scale       Scale one or multiple replicated services				扩容
>  update      Update a service														更新



集群服务启动：

>docker service create -p 8888:80 --name mynginx nginx			



>[root@iZm5eg01cs1709valycqzzZ ~]# docker service ps mynginx
>ID             NAME        IMAGE          NODE                      DESIRED STATE   CURRENT STATE                ERROR     PORTS
>v956zw11aslm   mynginx.1   nginx:latest   iZm5eg01cs1709valycqzzZ   Running         Running about a minute ago             
>[root@iZm5eg01cs1709valycqzzZ ~]# docker service ls
>ID             NAME      MODE         REPLICAS   IMAGE          PORTS
>c2uw10n297ej   mynginx   replicated   1/1        nginx:latest   *:8888->80/tcp



启动容器

docker service create --mode replicated/global -d -p 80:80 --name hzyy nginx

--mode  replicated / global 

replicated：默认，启动的容器随机在管理节点或者工作节点

global：启动的容器只能随机在管理节点上

动态扩/缩容

docker service update --replicas 3 mynginx

docker service scale 名称=个数		效果和update一样

移除服务

docker service rm 名称

# 其他命令

## 删除所有stop容器

```shel
docker container prune -f
```

## 复制本地文件到docker容器中

>如果是文件夹就用`文件夹位置/.`不需要加`-r` 

```shell
docker cp 文件位置 容器id:容器内文件位置
```



# 其他设置

## 设置代理

```shell
vim /etc/docker/daemon.json

写入:
https://dockerproxy.cn/
https://dockerpull.com/

sudo systemctl daemon-reload #重载systemd管理守护进程配置文件
sudo systemctl restart docker #重启 Docker 服务
```

# docker原理

>cgroups、namespaces和AUFS都是Linux操作系统中的重要功能，用于实现容器化和资源管理。
>
>1. **cgroups（Control Groups）**：cgroups是Linux内核中的一个功能，用于限制、控制和分配系统资源（如CPU、内存、磁盘I/O等）。它允许你将系统资源分组，然后为每个组分配特定的资源限制。这对于容器化应用程序至关重要，因为它允许你确保容器只使用其被分配的资源，从而避免一种应用程序耗尽系统资源而影响其他应用程序的情况发生。
>2. **Namespaces**：Linux namespaces是一种内核功能，用于隔离进程的视图，使它们看起来像是独立运行在一个独立的系统中。它们允许在同一系统上运行多个进程实例，每个实例都有自己的独立环境，包括进程ID、网络、文件系统等。这种隔离使得容器可以在同一台物理机上运行，但彼此之间彼此隔离，就像是在独立的虚拟机上运行一样。
>3. **AUFS（Another Union File System）**：AUFS是一种联合文件系统，它允许将多个文件系统联合挂载到单个目录中，使得这些文件系统的内容像是合并在一起的。这种技术对于容器化应用程序非常有用，因为它允许容器在运行时通过叠加文件系统层来修改其文件系统，而不需要复制整个文件系统。这样，容器可以共享基础文件系统的内容，并且可以轻松地进行修改和定制。
