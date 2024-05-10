[toc]

## Docker的三个概念

* 镜像（Image）：类似于虚拟机中的镜像，是一个包含有文件系统的面向Docker引擎的只读模板。任何应用程序运行都需要环境，而镜像就是用来提供这种运行环境的。例如一个Ubuntu镜像就是一个包含Ubuntu操作系统环境的模板，同理在该镜像上装上Apache软件，就可以称为Apache镜像。
* 容器（Container）：类似于一个轻量级的沙盒，可以将其看作一个极简的Linux系统环境（包括root权限、进程空间、用户空间和网络空间等），以及运行在其中的应用程序。Docker引擎利用容器来运行、隔离各个应用。容器是镜像创建的应用实例，可以创建、启动、停止、删除容器，各个容器之间是是相互隔离的，互不影响。注意：镜像本身是只读的，容器从镜像启动时，Docker在镜像的上层创建一个可写层，镜像本身不变。
* 仓库（Repository）：类似于代码仓库，这里是镜像仓库，是Docker用来集中存放镜像文件的地方。注意与注册服务器（Registry）的区别：注册服务器是存放仓库的地方，一般会有多个仓库；而仓库是存放镜像的地方，一般每个仓库存放一类镜像，每个镜像利用tag进行区分，比如Ubuntu仓库存放有多个版本（12.04、14.04等）的Ubuntu镜像。

![info](.\assets\info.png)



## 对镜像（Image）的基本操作

**创建镜像**

* 下载已有的镜像：`docker pull`

  ```bash
  docker pull centos
  ```

​	不加tag的话，取的是latest镜像



* 启动container并进行修改 -> commit：

  ```bash
  docker commit -m "centos with git" -a "sjj" c3e88982b536 sjj/centos:git
  docker images
  ```

  ![image-20221111225453140](.\assets\image-20221111225453140.png)

​		`-m`：指定说明信息

​		`-a`：指定用户信息

​		`c3e88982b536`：container id

​		`sjj/centos:git`：image的用户名、仓库名、tag



* 使用Dockerfile

  bash:

  ```bash
  docker build -t="sjj/centos:git_dockerfile" .
  ```

  `-t`指定新镜像的用户信息、tag，`.`表示在当前目录寻找dockerfile

  dockerfile:

  ```dockerfile
  # 说明该镜像以哪个镜像为基础
  FROM centos:centos7
  
  # 构建者的基本信息
  MAINTAINER xianhu
  
  # 在build这个镜像时执行的操作
  RUN yum -y update
  RUN yum install -y git
  
  # 拷贝本地文件到镜像中
  COPY ./* /usr/share/gitdir/
  ```

  ![image-20221112000807169](.\assets\image-20221112000807169.png)

​	

**删除镜像、容器**

```bash
docker rmi <image_name/image_id>
docker rm <container_name/container_id>
```



## 对容器（Container）的基本操作

**基于image启动container**

```bash
docker run -it centos:latest /bin/bash
```

`-i`：打开并保持stdout

`-t`：分配一个终端

`-d`：后台运行

后台运行时的stdout都输出到log，使用`docker logs <container_name/container_id>`查看

![image-20221112002833771](.\assets\image-20221112002833771.png)



**重启、停止container**

```bash
docker stop <container_name/container_id>
docker restart <container_name/container_id>
```



**进入一个后台启动的容器**

```bash
docker attach ...
```

