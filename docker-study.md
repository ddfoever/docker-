---
typora-copy-images-to: img
---

### Docker 学习

#### 1. Docker概念

+ Docker是一个开源应用容器引擎。
+ 完全使用沙箱机制，互相隔离
+ 容器性能极低

```小结：是一种容器技术，解决软件跨环境迁移技术```

##### 1.1 dokcer 安装

  ```shell
# 1.apt包更新到最新
$ apt update

# 安装包允许apt通过https使用仓库
$ sudo dpkg --configure -a
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

#添加Docker官方GPG key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#设置Docker稳定版仓库
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

#更新apt源索引
$ sudo apt-get update

#安装docker
$ sudo apt-get install docker-ce

  ```

##### 1.2 docker 容器镜像加速

使用阿里云镜像

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://x6wqiczq.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



#### 2. Docker 命令

##### 2.0 容器

```shell
# 容器重启
$ systemctl restart docker

# docker守护进程重新加载
$ systemctl daemon-reload

# 启动docker
$ systemctl start docker

# 查看docker状态
$ systemctl status docker

#开机启动docker
$ systemctl enable docker
```



##### 2.1 镜像

```shell
# 查看本地镜像
$ docker images

# 搜索网络中的镜像
$ docker search [name] //docker search redis

# 下载镜像 如果不写版本号 默认下载最新 Latest
$ docker pull [name] //  如果有版本号比如 docker pull redis:3.2 

# 删除某个docker 镜像
$ docker rmi [image_ID] 

#或者通过版本号删除 docker rmi [image_name]:[tag]
$ docker rmi redis:5.0

# 查询所有镜像列表的image_id
$ docker images -q

# 如果要删除所有的docker镜像
$ docker rmi `docker images -q`





```

##### 2.2 容器命令

```shell
# 查看容器
$ docker ps # 查看正在运行的容器
$ docker ps -a #查看所有容器

# 创建并且运行容器
$ docker run -it --name=c1 centos:7 /bin/bash 
$ docker run -id --name=c2 centos:7 /bin/bash

# 退出容器
$ exit

# 停止容器
$ docker stop [name/id]

# 启动容器
$ docker start [name/id]

# 进入容器
$ docker exec -it [name/id] /bin/bash

# 删除容器
$ docker rm [id/name]

# 查看所有容器的id
$ docker ps -aq

# 删除所有容器
$ docker rm `docker ps -aq`  #不能停止正在运行的容器
$ docker rm c1 c2 c3 c4

# 查看容器信息
$ docker inspect [name/id]


```

> 参数说明：

+ -i 保持容器运行。通常与-t同时使用，加入it这两个参数后，容器创建后 自动进入容器内部，退出容器后，容器也自动关闭
+ -t 为容器重新分配一个伪终端，通常与-i使用
+ -d 以守护模式运行容器，创建一个容器在后台运行，需要使用docker exec进入容器，退出后容器不会关闭。

##### 2.3 容器数据卷

###### 2.3.1 数据卷概念及作用

	> 思考： 1.docker 容器删除后，容器产生的数据还在吗？ **不在了**
	>
	> 2. doker容器和外部机器可以直接交换文件吗 **不可以**
	> 3. 容器之间想要进行数据交互？

  <font color='red'>**数据卷**</font>

+ **数据卷是宿主机中的一个目录或者文件**
+ **当容器目录和数据卷绑定后，对方的修改会立即同步**

<img src="https://raw.githubusercontent.com/ddfoever/docker-learning/master/img/image-20210114220116315.png" alt="image-20210114220116315" style="zoom:50%;" />



**一个容器可以挂载多个数据卷，多个容器也可以挂载一个数据卷**

###### 2.3.2  配置数据卷

+ 创建容器时，使用**-v**参数 设置数据卷 **echo  neirong->文件**

  ```sh
  $ docker run -it --name=C1 -v /root/data:/root/data Centos:7 /bin/bash
  ```

  <font color='red'>**注意：**</font>

  > 1. **`/root/data`** 必须是绝对路径 使用`/` 或者`~/ ` `-v 宿主机目录(文件)：容器内目录(文件)` ,  `~/` 只能在宿主机中使用
  > 2. 可以挂载多个数据卷
  > 3. 如果目录不存在，则自动创建

+ 挂载多个数据卷

  ```shell
  $ docker run -it --name=C2 -v ~/data1:/root/data1 \
  > -v ~/data2:/root/data2 \
  > Centos:7 /bin/bash
  ```

  <font color='red'> **注意：**</font>

  `\` **使用可以换行**

###### 2.3.3 配置数据卷容器

> 问题：多容器进行数据交换  有几种方式
>
> 1. 多个容器挂载同一个数据卷（容器多了的话， 比较麻烦）
> 2. 数据卷容器

<img src="https://raw.githubusercontent.com/ddfoever/docker-learning/master/img/image-20210114224137052.png" alt="image-20210114224137052" style="zoom:50%;" />

+ 创建启动C3数据卷容器，使用-v参数，设置数据卷

  ```SHELL
  $ docker run -it --name=C3 -v /volume centos:7 /bin/bash
  ```

+ 创建c1 c2 容器，使用--volumes-from 参数设置数据卷

       ```SHELL
  $ docker run -it --name=c1 --volumes-from c3 centos:7 /bin/bash
       ```

      ```SHELL
  $ docker run -it --name=c2 --volumes-from c3 centos:7 /bin/bash
      ```

> **小结：**
>
> 1. **数据卷概念**
>    + **宿主机的一个目录或者文件**
> 2. **数据卷的作用**
>    - **容器的数据持久化**
>    - **客户端和容器数据交换**
>    - **容器件数据交换**
> 3. **数据卷容器**
>    + **创建一个容器，挂载一个目录，让其他容器继承该容器（--volumes-from）**
>    + **通过简单方式实现数据卷配置** 	

##### 2.4 Docker应用部署

###### 2.4.1 myslq 部署

 ```shell
# 创建mysql 容器，并且创建相应的mysql 目录
$ cd ~
$ mkdir mysql
$ cd mysql

# 创建myslq 容器并且和宿主机相互映射
$ docker run -it --name=c_mysql \
-p 3306:3306 \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD:root mysql:5.7

 ```

###### 2.4.2 Tomcat 部署

###### 2.4.3 Ngnix部署

###### 2.4.4 Redis部署

##### 2.5 dockerfile

###### 2.5.1 dokcer原理

> 思考：
>
> 1. docker镜像的本质是什么？ <font color='red'>一个分层的文件系统</font>
>
>    ```markdown
>    ##### dokcer镜像原理
>    操作系统组成部分：
>     1. 进程调度子系统
>     2. 进程通信子系统
>     3. 内存管理子系统
>     4. 设备管理子系统
>     5. 文件管理子系统
>     6. 网络通信子系统
>     7. 作业控制子系统
>    ```
>
>    <img src="img/image-20210116220219158.png" alt="image-20210116220219158" style="zoom:50%;" />
>
>    ```markdown
>     ### 其中Linux文件系统bootfs和rootfs两部分组成
>     + bootfs: 包含了bootloader（引导加载程序）和kernel（内核）
>     + rootfs: root文件系统，包含的就是经典Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件
>     + 不同的linux发行版本，bootfs基本一样，而rootfs不同，如ubuntu，centos等。
>    ```
>
>    ```markdown
>    #### docker镜像是由特殊的文件系统叠加而成
>    + 最低端是bootfs,并使用宿主机的rootfs
>    + 第二层root文件系统rootfs，称为base image
>    + 然后再往上可以叠加其他镜像文件
>    + 统一文件系统技术能够将不同的层整合成一个文件系统，为这些系统提供一个统一的视角，这样就隐藏了多层的存在，在用户的角度来看，只存在一个文件系统
>    + 一个镜像可以放在另一个镜像上面，位于下面的镜像称为父镜像，最底部的成为基础镜像
>    + 当一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器
>    ```
>
>    <img src="img/image-20210116221815408.png" alt="image-20210116221815408" style="zoom:50%;" />
>
>    1. docker中一个centos镜像为什么只有200mb，而一个centos操作系统的iso文件要几个G    复用了centos操作系统的bootfs，只有rootfs和其他镜像层。
>
> 2. Docker中一个tomcat镜像有个500MB，而一个tomcat安装包只有70MB?  

###### 2.5.2 概念作用

###### 2.5.3 镜像制作

    ###### docker 镜像制作

```shell
#1. 容器转镜像
$ docker commit [container_id] [name]:[version]
$ docker save -o xxx.tar
$ docker load -i xxx.tar

```

<img src="img/image-20210116230404699.png" alt="image-20210116230404699" style="zoom:50%;" />

````shell
# dockerfile
# 1.Dockerfile 是一个文本文件
# 2.包含了一条条的指令
# 3.每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像


````

###### 2.5.4 自定义centos

```shell
案例：需求
自定义centos7镜像，要求：
1. 默认登录 路径为/usr
2. 可以使用vim
案例：实现步骤
1. 实现父镜像：FROM centos：7
2.定义作者信息 MAINTAINER itan
3. 执行安装命令 RUN yum install -y vim
4. 定义默认的工作目录： WORKDIR /usr
5. 定义容器启动执行的命令： CMD /bin/bash

```

```shell
# dockerfile 的构建镜像的命令
$ docker build -f [dockerfile 的路径] -t [镜像名称:version] .   //不加[镜像名称:version] 表示latest ，最新版本
```

###### 2.5.5  docker部署springboot 项目

```shell
# 创建一个springboot项目 并且打包上传到linux 服务器
# 使用有权限的tmp 文件夹， 并且切换到root账户， mv到需要的文件夹
```

```shell
# 案例：定义dockerfile 发布springboot项目

# 案例：实现步骤：
# 1. 定义父镜像，FROM java:8
2.   定义作者信息  MAINTAINER itan
3. 添加jar包到容器中  ADD  springboot.jar app.jar
4.  CMD java -jar
```

##### 2.6 docker 服务编排

###### 2.6.1 服务编排概念

    ```markdown
# 服务编排
微服务架构的应用系统中一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，维护工作量很大。
1. 要从dockerfile build image或者去dockerhub 拉去image
2. 要创建多个container
3. 要管理这些container（启动停止删除）
    ```

<font color='red'>**服务编排：按照一定的业务规则批量管理容器**</font>

###### 2.6.2  Docker compose 概述

 ```markdown
docker compose 是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止，使用步骤：
1. 利用Dockerfile定义运行环境镜像
2. 使用docker-compose.yml定义组成应用的各服务
3. 运行docker-compose up 启动
 ```

<img src="img/image-20210117222435292.png" alt="image-20210117222435292" style="zoom:50%;" />

###### 2.6.3 docker-compse 安装

````shell
#	Compose 目前已经完全支持Linux、mac、windows,在我们安装Compose之前，需要先安装docker
$ curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /user/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ docker-compose -version
````

```shell
# 另一种方式
$ curl -L https://github.com/docker/compose/releases/download/1.22.0/run.sh > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ docker-compose -v
```

<font color='red'>**chmod +x的意思就是给执行权限**</font>

###### 2.6.4 删除docker-compose

```shell
# 二进制包安装的，删除二进制文件即可
$ rm /usr/local/bin/docker-compose
```

###### 2.6.5 案例



```shell
# 使用ngnix+springboot 编排部署容器
1.
$ cd /usr
$ mkdir docker-compose
$ cd docker-compose
$ vim docker-compose.yml


```

2. **创建docker-compose.yml**

   ```shell
   version: '3'
   services:
     nginx:
       image: nginx
       ports:
         - 80:80
       links:
         - app
       volumes:
         - ./nginx/conf.d:/etc/nginx/conf.d
     app:
       image: app
       expose:
         - ="8080"
         
         
   version: '3'
   services:
     nginx:
       image: nginx:latest
       ports:
         - 80:80
       links:
         - app
       volumes:
         - ./nginx/conf.d:/etc/nginx/conf.d
     app:
       image: app:1
       expose:
         - "8080"
   
         
   ```

3. **创建./nginx/conf.d目录**

   ```shell
   mkdir -p ./nginx/conf.d
   ```

4. **在./nginx/conf.d目录下面创建itan.conf(nginx 的配置文件名随便起，但是后缀必须是.conf)**

   ```shell
   ### 配置app的反向代理
   server {
     listen 80;
     access_log off;
     location / {
      proxy_pass http://app:8080;
     }
   }
   
   ```

5. **使用docker-compose up启动** 

##### 2.7 docker 私有仓库搭建



