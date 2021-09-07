[toc]
---

# Docker简介

## Docker应用场景

Web 应用的自动化打包和发布。

- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。  

## Docker优点
Docker是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用Docker的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。  
### 1、快速，一致地交付您的应用程序
Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。容器非常适合持续集成和持续交付（CI / CD）工作流程，请考虑以下示例方案：  
- 您的开发人员在本地编写代码，并使用 Docker 容器与同事共享他们的工作。
- 他们使用 Docker 将其应用程序推送到测试环境中，并执行自动或手动测试。
- 当开发人员发现错误时，他们可以在开发环境中对其进行修复，然后将其重新部署到测试环境中，以进行测试和验证。
- 测试完成后，将修补程序推送给生产环境，就像将更新的镜像推送到生产环境一样简单。

### 2、响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。

Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

### 3、在同一硬件上运行更多工作负载
Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

## 为何使用Docker?
- Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性，从而不会再出现 “这段代码在我机器上没问题啊” 这类问题；——一致的运行环境
- 可以做到秒级、甚至毫秒级的启动时间。大大的节约了开发、测试、部署的时间。——更快速的启动时间
- 避免公用的服务器，资源会容易受到其他用户的影响。——隔离性
- 善于处理集中爆发的服务器使用压力；——弹性伸缩，快速扩展
- 可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。——迁移方便
- 使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。——持续交付和部署


## Docker架构
Docker 包括三个基本概念:
- 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- 仓库（Repository）：仓库可看成一个代码控制中心，用来保存镜像。  

Docker 使用客户端-服务器 (C/S)架构模式，使用远程API来管理和创建Docker容器。Docker 容器通过 Docker 镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。
![image](https://www.runoob.com/wp-content/uploads/2016/04/576507-docker1.png)

| 概念 | 说明 |
| ---- | ---- |
|Docker 镜像(Images) | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。|
|Docker 容器(Container)| 容器是独立运行的一个或一组应用，是镜像运行时的实体。|
|Docker 客户端(Client)|Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。|
|Docker 主机(Host)| 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。|
|Docker Registry|Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。|
|Docker Machine|Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。|

### 镜像（Image）- 一个特殊的文件系统

**操作系统分为内核和用户空间**。对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而Docker 镜像（Image），就相当于是一个 root 文件系统。

**Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）**。 镜像不包含任何动态数据，其内容在构建之后也不会被改变。

Docker 设计时，就充分利用 **Union FS**的技术，将其设计为**分层存储**的架构 。 镜像实际是由多层文件系统联合组成。

**镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层**。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

### 容器（Container）- 镜像运行时的实体

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，**容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等**。

**容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。前面讲过镜像使用的是分层存储，容器也是如此**。

**容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失**。

按照 Docker 最佳实践的要求，**容器不应该向其存储层内写入任何数据** ，容器存储层要保持无状态化。**所有的文件写入操作，都应该使用数据卷（Volume）、或者绑定宿主目录**，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储)发生读写，其性能和稳定性更高。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此， **使用数据卷后，容器可以随意删除、重新 run ，数据却不会丢失**。

### 仓库（Repository）- 集中存放镜像文件的地方

镜像构建完成后，可以很容易的在当前宿主上运行，但是， **如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry就是这样的服务**。
一个 Docker Registry中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。所以说：**镜像仓库是Docker用来集中存放镜像文件的地方类似于我们之前常用的代码仓库**。
通常，**一个仓库会包含同一个软件不同版本的镜像**，而标签就常用于对应该软件的各个版本 。我们可以通过`<仓库名>:<标签>`的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签.。

## Docker 中的Build, Ship, and Run

- Build（构建镜像） ： 镜像就像是集装箱包括文件以及运行环境等等资源。
- Ship（运输镜像） ：主机和仓库间运输，这里的仓库就像是超级码头一样。
- Run （运行镜像） ：运行的镜像就是一个容器，容器就是运行程序的地方。

Docker 运行过程也就是去仓库把镜像拉到本地，然后用一条命令把镜像运行起来变成容器。所以，我们也常常将Docker称为码头工人或码头装卸工，这和Docker的中文翻译搬运工人如出一辙。

## Docker VS VM
传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。
|概念 | 概念 |说明|
|:---:|:---:|:---:|
|启动|秒级|分钟级|
|硬盘使用|一般为MB|一般为GB|
|性能|接近原生|弱于|
|系统支持量|单机支持上千个容器|一般支持几十个|

- 容器是一个应用层抽象，用于将代码和依赖资源打包在一起。 多个容器可以在同一台机器上运行，共享操作系统内核，但各自作为独立的进程在用户空间中运行 。与虚拟机相比， 容器占用的空间较少（容器镜像大小通常只有几十兆），瞬间就能完成启动 。
- 虚拟机 (VM) 是一个物理硬件层抽象，用于将一台服务器变成多台服务器。 管理程序允许多个 VM 在一台机器上运行。每个VM都包含一整套操作系统、一个或多个应用、必要的二进制文件和库资源，因此 占用大量空间 。而且 VM  启动也十分缓慢 。

## Docker 安装教程

### linux CentOS 离线安装docker 一

> 需要root权限或者是sudo

1. 下载静态二进制存档。转到 https://download.docker.com/linux/static/stable/ （或更改stable为nightly或test），选择您的硬件平台，然后下载.tgz与要安装的Docker Engine版本有关的文件。
2. 使用该tar命令解压文件
```shell
$ tar zxvf /path/to/<FILE>.tar.gz
```
3. 将二进制文件移至可执行路径上的目录(/usr/bin/)下
```shell
$ sudo cp docker/* /usr/bin/
或者在root用户下
[root]# cp docker/* /usr/bin/
```
4. 启动Docker守护程序：
```shell
$ sudo dockerd &
或者在root用户下
[root]# dockerd &
```
如果需要使用其他选项启动守护程序，请相应地修改以上命令，或者创建并编辑文件/etc/docker/daemon.json 以添加定制配置选项。

5. 可以尝试着打印下版本号，试着看看 images，看看 info，看看容器了
```shell
$ sudo docker images
$ sudo docker ps -a
$ sudo docker --version
$ sudo docker info
或者在root用户下
[root]# docker images
[root]# docker ps -a
[root]# docker --version
[root]# docker info
```
通过运行hello-world 映像来验证是否正确安装了Docker`<hostname>`是运行Docker守护程序并可供客户端访问的主机名或IP地址。
```shell
$ sudo docker [-H <hostname>] run hello-world
或者在root用户下
[root]# docker [-H <hostname>] run hello-world
```
### linux CentOS 离线安装docker 二

1. 下载静态二进制存档。转到 https://download.docker.com/linux/static/stable/ （或更改stable为nightly或test），选择您的硬件平台，然后下载.tgz与要安装的Docker Engine版本有关的文件。
2. 使用该tar命令解压文件
```shell
$ tar zxvf /path/to/<FILE>.tar.gz
```
3. 将二进制文件移至可执行路径上的目录(/usr/bin/)下
```shell
$ sudo cp docker/* /usr/bin/
或者在root用户下
[root]# cp docker/* /usr/bin/
```
4. 注册服务

将docker注册为service
```shell
$ vim /etc/systemd/system/docker.service
```
复制下面的内容到docker.service
```
[Unit]
 
Description=Docker Application Container Engine
 
Documentation=https://docs.docker.com
 
After=network-online.target firewalld.service
 
Wants=network-online.target
 
[Service]
 
Type=notify
 
# the default is not to use systemd for cgroups because the delegate issues still
 
# exists and systemd currently does not support the cgroup feature set required
 
# for containers run by docker
 
ExecStart=/usr/bin/dockerd
 
ExecReload=/bin/kill -s HUP $MAINPID
 
# Having non-zero Limit*s causes performance problems due to accounting overhead
 
# in the kernel. We recommend using cgroups to do container-local accounting.
 
LimitNOFILE=infinity
 
LimitNPROC=infinity
 
LimitCORE=infinity
 
# Uncomment TasksMax if your systemd version supports it.
 
# Only systemd 226 and above support this version.
 
#TasksMax=infinity
 
TimeoutStartSec=0
 
# set delegate yes so that systemd does not reset the cgroups of docker containers
 
Delegate=yes
 
# kill only the docker process, not all processes in the cgroup
 
KillMode=process
 
# restart the docker process if it exits prematurely
 
Restart=on-failure
 
StartLimitBurst=3
 
StartLimitInterval=60s
 
[Install]
 
WantedBy=multi-user.target
```
5. 启动
```shell
$ chmod +x /etc/systemd/system/docker.service # 添加文件权限

$ systemctl daemon-reload # 重载unit配 置文件

$ systemctl start docker # 启动Docker

$ systemctl enable docker.service # 设置开机自启
```
6. 验证
```shell
$ systemctl status docker # 查看Docker状态
$ docker -v # 查看Docker版本
$ docker images
$ docker ps -a
$ docker --version
$ docker info
```

---

# Docker常用命令

> 使用命令可以参考参考文献中的网站三

**介绍一些常用的docker命令:**

1. **启动容器** 
docker run -it 镜像 命令
```shell
$ docker run -it ubuntu /bin/bash
```
> 参数介绍：<br>
> &nbsp;&nbsp; -i: 交互式操作。<br>
> &nbsp;&nbsp; -t: 终端。<br>
> &nbsp;&nbsp; ubuntu: ubuntu 镜像。<br>
> &nbsp;&nbsp; /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。<br>

2. **启动已停止运行的容器**
docker start 容器ID
```shell
$ docker start b750bbbcfd88 
```
> 查看所有的容器命令如下：
> &nbsp;&nbsp;docker ps -a 
> 查看正在运行的命令如下：
> &nbsp;&nbsp;docker ps 

3. **停止已启动的容器**
docker stop 容器ID
```shell
$ docker stop b750bbbcfd88 
```

4. **重新启动的容器**
docker restart 容器ID
```shell
$ docker restart b750bbbcfd88 
```

5. **后台运行一个容器**
docker run -d 容器ID
```shell
$ docker run -itd --name ubuntu-test ubuntu /bin/bash
```
> 加了 -d 参数默认不会进入容器，想要进入容器需要使用指令 docker exec

6. **进入一个容器**
- docker attach
- docker exec：推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

```shell
$ docker attach 1e560fca3906 
$ docker exec -it 243c32535da7 /bin/bash
```
7. **导入/导出容器**
```shell
# 导出容器
$ docker export 1e560fca3906 > ubuntu.tar
# 导入容器
$ cat docker/ubuntu.tar | docker import - test/ubuntu:v1
# 通过url或者某个目录来导入
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```
8. **删除容器**
docker rm -f 容器ID
```shell
$ docker rm -f 1e560fca3906
```

9. **运行一个web应用**
```shell
$ docker pull training/webapp  # 载入镜像
$ docker run -d -P training/webapp python app.py
```
> 参数说明：
> &nbsp;&nbsp;-d:让容器在后台运行。
> &nbsp;&nbsp;-P:将容器内部使用的网络端口随机映射到我们使用的主机上。

**Docker镜像管理**
1. **列出镜像列表**
```shell
$ docker images
```
> 各个选项说明:
> &nbsp;&nbsp;REPOSITORY：表示镜像的仓库源
> &nbsp;&nbsp;TAG：镜像的标签,用来进行版本控制
> &nbsp;&nbsp;IMAGE ID：镜像ID
> &nbsp;&nbsp;CREATED：镜像创建时间
> &nbsp;&nbsp;SIZE：镜像大小

2. **获取镜像**
docker pull 镜像
```shell
$ docker pull ubuntu
```
3. **删除镜像**
docker rmi 镜像
```shell
$ docker rmi ubuntu
```

4. **保存一个镜像到本地**
docker save -o 本地文件名.docker 镜像名称:tag
```shell
$ docker save -o java_jdk.docker java:8
```
5. **从本地加载一个镜像**
docker load -i 本地文件名.docker
```shell
$ docker load -i java_jdk.docker
```

**Docker容器连接**
1. **网络端口映射**
容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P 或 -p 参数来指定端口映射。
**两种方式的区别是:**
- -P :是容器内部端口随机映射到主机的高端口。
- -p : 是容器内部端口绑定到指定的主机端口。

```shell
$ docker run -d -P training/webapp python app.py
$ docker run -d -p 5000:5000 training/webapp python app.py
```
> docker port 命令可以让我们快捷地查看端口的绑定情况。

**Docker 容器互联**
端口映射并不是唯一把 docker 连接到另一个容器的方法。
docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。
docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。

2. **容器命名**
当我们创建一个容器的时候，docker 会自动对它进行命名。另外，我们也可以使用 --name 标识来命名容器
```shell
$ docker run  --name runoob hello-world
```

3. **新建网络**
```shell
$ docker network create -d bridge test-net
```
> 参数说明：
> &nbsp;&nbsp;-d：参数指定 Docker 网络类型，有 bridge、overlay。

4. **查看docker的网络**
```shell
$ docker nerwork ls
```

5. **连接容器**
运行一个容器并连接到新建的 test-net 网络:
```shell
$ docker run -itd --name test1 --network test-net ubuntu /bin/bash
```

6. **配置 DNS**
可以在宿主机的 `/etc/docker/daemon.json`文件中增加以下内容来设置全部容器的 DNS：
```json
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```
> 设置后，启动容器的 DNS 会自动配置为 114.114.114.114 和 8.8.8.8。  
> 配置完，需要重启 docker 才能生效。

手动指定容器的配置:
```shell
$ docker run -it --rm host_ubuntu  --dns=114.114.114.114 --dns-search=test.com ubuntu
```
> 参数说明：
> &nbsp;&nbsp;-h HOSTNAME 或者 --hostname=HOSTNAME： 设定容器的主机名，它会被写到容器内的 /etc/hostname 和 /etc/hosts。
> &nbsp;&nbsp;--dns=IP_ADDRESS：添加 DNS 服务器到容器的 /etc/resolv.conf 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。
> &nbsp;&nbsp;--dns-search=DOMAIN：设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com。

---

# Dockerfile

>  使用Dockerfile构建镜像

docker build -t 新名称:tag -f Dockerfile的名称 .
```shell
$ docker build -t second:v1.0 -f Dockerfile . 
# -t 给镜像取一个名称
# -f 指定构建镜像的 Dockerfile 文件（Dockerfile 可不在当前路径下）；不指定时默认为当前文件下下的Dockerfile文件，注意大小写。
# . 为上下文路径，指定构建镜像的上下文的路径，构建镜像的过程中，可以且只可以引用上下文中的任何文件 。
```

1. **From和Run指令**
FROM：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。  
RUN：用于执行后面跟着的命令行命令。有以下俩种格式：
```dockerfile
# shell 格式：
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
# exec 格式：
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```
> Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。

2. **上下文路径**
上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。
**解析：** 由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。
如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。

> 上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。

3. **COPY 指令**
复制指令，从上下文目录中复制文件或者目录到容器里指定路径。
```dockerfile
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```
> [--chown=<user>:<group>]：可选参数，用户改变复制到容器内文件的拥有者和属组。
> <源路径>：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：
> &nbsp;&nbsp;`COPY hom* /mydir/`
> &nbsp;&nbsp;`COPY hom?.txt /mydir/`
> <目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

4. **ADD指令**
ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：
- ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
- ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。

5. **CMD指令**
类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
`CMD 在docker run 时运行。`
`RUN 是在 docker build。`
**作用：** 为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。
**注意：** 如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。
```dockerfile
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```
> 推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

6. **ENTRYPOINT指令**
类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。
但是, 如果运行 docker run 时使用了 --entrypoint 选项，此选项的参数可当作要运行的程序覆盖 ENTRYPOINT 指令指定的程序。
**优点：** 在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。
**注意：** 如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。
```dockerfile
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```
> 可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT。

7. **ENV指令**
设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。
```dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

8. **ARG指令**
构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。
构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。
```dockerfile
ARG <参数名>[=<默认值>]
```

9. VOLUME指令
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。
作用：  
- 避免重要的数据，因容器重启而丢失，这是非常致命的。  
- 避免容器不断变大。
```dockerfile
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```
> 在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

10. **EXPOSE指令**
仅仅只是声明端口。
作用：
- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。
```dockerfile
EXPOSE <端口1> [<端口2>...]
```
11. **WORKDIR指令**
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。
docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。
```dockerfile
WORKDIR <工作目录路径>
```

12. **USER指令**
用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。
```dockerfile
USER <用户名>[:<用户组>]
```

13. **HEALTHCHECK指令**
用于指定某个程序或者指令来监控 docker 容器服务的运行状态。
```dockerfile
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```
14. **ONBUILD指令**
用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这是执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。
```dockerfile
ONBUILD <其它指令>
```

---

# docker 搭建私人镜像仓库

需要准备一台机器作为私人仓库服务器假设地址为192.168.72.200；需要服务端（要使用镜像的机器），假设地址为182.168.72.201。
**服务器端需要的操作：**
1. 从可以联网的docker上pull registry
```shell
$ docker pull registry:2
```
2. 保存镜像为归档文件
```shell
$ docker save -o registry.tar registry:2
```
3. 上传镜像到内网docker
```shell
$ docker load -i  registry.tar 
```
4. 启动容器
```shell
$ docker run -d -v /root/docker/registry:/var/lib/registry -p 5000:5000 --name myregistry --restart=always registry:2
```
**客户端需要的操作：**
1. 配置私有仓库地址
```json
vim /etc/docker/daemon.json # 严格的json文件格式
加入以下内容：
{
"insecure-registries": ["192.168.72.200:5000"] # 私有仓库服务器的地址和端口（-p映射的端口）
}
```
2. 重启docker服务
```shell
$ systemctl daemon-reload # 重载unit配置文件

$ systemctl restart docker # 重新启动Docker
```
**使用私有仓库**

1. push 操作
```shell
# 修改镜像标签为服务器地址:端口/镜像名:tag；docker会按照标签的地址和端口推送镜像
$ docker tag busybox:latest 192.169.72.200:5000/busybox:v0.1

# 上传镜像
$ docker push 192.169.72.200:5000/busybox:v0.1
```
2. pull 操作
```shell
# 镜像为服务器地址:端口/镜像名
$ docker pull 192.169.72.200:5000/busybox:v0.1
```
3. 查询镜像
```shell
# 服务器的地址和端口
$ curl http://192.169.72.200:5000/v2/_catalog

# 查询镜像版本 curl http://your-server-ip:5000/v2/your-image-name/tags/list
$ curl http://192.169.72.200:5000/v2/busybox/tags/list
```

---

# References(参考文献)

网址一:https://www.runoob.com/docker/docker-tutorial.html 
网站二:https://juejin.im/post/6844903625584558093#heading-12 
网站三:https://www.runoob.com/docker/docker-command-manual.html 