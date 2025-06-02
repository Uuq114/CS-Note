# 深入剖析 Kubernetes

## Docker

### Docker 背景

Docker 之前的 PaaS 产品，Cloud Foundary：

- 一套应用的打包和分发机制
- 为每种编程语言定义一种打包格式，将可执行文件和启动脚本打包
- 为每个应用创建一个 “沙盒” 的隔离环境

Cloud Foundary：用户就必须为每种语言、每种框架，甚至每个版本的应用维护一个打好的包

Docker 成功的关键是：镜像。
大多数 Docker 镜像是操作系统 + 应用 + 脚本，因此可以保证本地、云端环境完全一致，不需要额外配置和修改

Docker 解决了环境一致性的问题，但是没有解决大规模部署应用的问题。“容器编排” 比容器本身更有价值。

### Docker 实现方法

容器是一种特殊的进程。操作系统在启动进程时通过设置一些参数，实现了资源隔离和限制

- cgroup：资源限制。
- namespace：隔离资源。Linux 提供了 PID、Mount、Network 等多种 namespace。应用进程只能看到指定的内容

Linux Cgroup，全程为 linux control group，用于限制进程组能使用的资源上限，例如 CPU、内存、磁盘、网络带宽等

cgroup 通过文件和目录的方式暴露接口，在 `/sys/fs/group` 目录下。

![alt text](img/image.png)

在对应的子系统新建一个目录，就新建了一个 “控制组”，系统会自动在这个目录下新建一些文件，表示限制的任务（PID，`tasks` 文件）、限制的量

### Docker 的局限

和虚拟机相比，基于 Linux namespace 的虚拟化技术主要问题是：隔离不彻底

- 多个容器使用的是同一宿主机的内核。无法在 windows 主机运行 Linux 容器
- 有的资源无法 namespace 化，例如时间

### Docker 容器镜像

Linux 提供了 `chroot` 命令，可以将进程的根目录改变到指定位置，mount namespace 就是在这个命令上发展的。

通过将一个完整 OS 的文件系统挂载到容器根目录，可以让容器看到的文件系统更真实。这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的 “容器镜像”，又称 rootfs（根文件系统）。

容器镜像（rootfs）只是 OS 的文件和目录，不包括内核，一般几百 MB。虚拟机的镜像大多是磁盘的快照。

```bash
$ ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
```

进入容器后执行的 `/bin/bash`，和宿主机的不是一个 bash

### Docker layer

在制作应用依赖环境时，我们希望能基于 rootfs 做增量修改，避免对每个应用都要重复制作 rootfs。因此，Docker 在设计镜像时引入了 layer 的概念。

用户制作镜像的每一个操作，都会生成一个 layer，也就是一个增量 rootfs。通过 UnionFS（可以将多个目录联合挂载到同一个目录）

```bash
# 从 Docker Hub 上拉取一个 Ubuntu 镜像到本地
$ docker run -d ubuntu:latest sleep 3600

# 查看 Docker 镜像 rootfs，发现由多个层组成。在使用镜像时，Docker 会把这些增量联合挂载在一个统一的挂载点上
$ docker image inspect ubuntu:latest
...
     "RootFS": {
      "Type": "layers",
      "Layers": [
        "sha256:f49017d4d5ce9c0f544c...",
        "sha256:8f2b771487e9d6354080...",
        "sha256:ccd4d61916aaa2159429...",
        "sha256:c01d74f99de40e097c73...",
        "sha256:268a067217b5fe78e000..."
      ]
    }
```

通过在 `/sys/fs/aufs` 记录的挂载信息，5 个镜像层被联合挂载成完整的 Ubuntu 文件系统

![alt text](img/image-1.png)

- 只读层（ro+wh）。包含 `/usr`、`/sbin` 等目录，以增量的方式包含 OS 的一部分
- 读写层。如果在容器里做了写操作，修改内容会以增量方式出现在这一层。如果是删除操作，会创建一个 whiteout 文件，将只读层的文件 “遮挡” 起来
- init 层。Docker 项目单独生成的内部层。保存 `/etc/hosts`、`/etc/resolv.conf` 等信息。为什么需要 init 层？因为用户需要修改这些信息，并且这些配置只对当前容器有效，不希望提交这些信息。（docker commit 只会提交读写层）

最终，这 7 个层都被联合挂载到 `/var/lib/docker/aufs/mnt` 目录下，表现为一个完整的 Ubuntu 操作系统供容器使用。