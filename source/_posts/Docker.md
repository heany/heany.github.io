---
title: Docker技术学习
date: 2018-04-16 15:20:03
categories:
- 分布式与大数据
tags:
- Docker
---

## 一、 前言
我们都知道软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？

用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。

如果某些老旧的模块与当前环境不兼容，那就麻烦了。开发者常常会说："它在我的机器可以跑了"（It works on my machine），言下之意就是，其他机器很可能跑不了。

环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。

虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。虽然用户可以通过虚拟机还原软件的原始环境，但是虚拟机却有很多缺点，资源占用较多，拖慢计算机的运行速度，
冗余步骤多;启动很慢等。结构对比如下图所示：

<img src="https://wx3.sinaimg.cn/mw1024/e0db46edgy1fqek1ao1k6j20kt0e8dhv.jpg" align="center">

由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。

Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程的外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。由于**容器**是进程级别的，相比虚拟机有很多优势。例如：启动很快，资源占用少;体积很小。容器就像是轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销比虚拟机要小得多。



## 二、 Docker是什么？

**Docker属于Linux容器的一种封装，提供简单易用的容器使用接口**。它是目前最流行的 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

## 三、 基本概念
Docker包括三个基本概念：

+ 镜像（Image）
+ 容器（Container）
+ 仓库（Repository）

理解了这三个概念，也就理解了Docker的整个生命周期。

### 3.1、 `Docker`镜像（Image）
我们都知道,操作系统分为内核和用户空间。对于Linux而言,内核启动后,会挂载root文件系统为其提供用户空间支持。而 Docker 镜像(Image),就相当于是一个文件系统。比如官方镜像`ubuntu:16.04`就包含了完整的一套`Ubuntu 16.04`最小系统的`root`文件系统。

Docker 镜像是一个特殊的文件系统,除了提供容器运行时所需的程序、库、资源、配置等文件外,还包含了一些为运行时准备的一些配置参数(如匿名卷、环境变量、用户等)。镜像不包含任何动态数据,其内容在构建之后也不会被改变。

#### 分层存储
因为镜像包含操作系统完整的`root`文件系统,其体积往往是庞大的,因此在`Docker`设计时,就充分利用`Union FS`的技术,将其设计为分层存储的架构。所以严格来说,镜像并非是像一个`ISO`那样的打包文件,镜像只是一个虚拟的概念,其实际体现并非由一个文件组成,而是由一组文件系统组成,或者说,由多层文件系统联合组成。

镜像构建时,会一层层构建,前一层是后一层的基础。每一层构建完就不会再发生改变,后一层上的任何改变只发生在自己这一层。比如,删除前一层文件的操作,实际不是真的删除前一层的文件,而是仅在当前层标记为该文件已删除。在最终容器运行的时候,虽然不会看到这个文件,但是实际上该文件会一直跟随镜像。因此,在构建镜像的时候,需要额外小心,每一层尽量只包含该层需要添加的东西,任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层,然后进一步添加新的层,以定制自己所需的内容,构建新的镜像。


### 3.2、 `Docker`容器
镜像(Image)和容器(Container)的关系,就像是面向对象程序设计中的类和实例一样,镜像是静态的定义,容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程,但与直接在宿主执行的进程不同,容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的`root`文件系统、自己的网络配置、自己的进程空间,甚至自己的用户`ID`空间。容器内的进程是运行在一个隔离的环境里,使用起来,就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性,很多人初学`Docker`时常常会混淆容器和虚拟机。

前面讲过镜像使用的是分层存储,容器也是如此。每一个容器运行时,是以镜像为基础层,在其上创建一个当前容器的存储层,我们可以称这个**为容器运行时读写而准备的存储层为容器存储层。**

容器存储层的生存周期和容器一样,容器消亡时,容器存储层也随之消亡。因此,任何保存
于容器存储层的信息都会随容器删除而丢失。

按照 Docker 最佳实践的要求,容器不应该向其存储层内写入任何数据,容器存储层要保持无
状态化。所有的文件写入操作,都应该使用数据卷(Volume)、或者绑定宿主目录,在这些位置的读写会跳过容器存储层,直接对宿主(或网络存储)发生读写,其性能和稳定性更高。

数据卷的生存周期独立于容器,容器消亡,数据卷不会消亡。因此,使用数据卷后,容器删除或者重新运行之后,数据却不会丢失。



### 3.3、 `Docker`仓库（Registry）
镜像构建完成后,可以很容易的在当前宿主机上运行,但是,如果需要在其它服务器上使用这个镜像,我们就需要一个集中的存储、分发镜像的服务,Docker Registry 就是这样的服务。

一个`Docker Registry`中可以包含多个仓库(`Repository`);每个仓库可以包含多个标签(`Tag`);每个标签对应一个镜像。

通常,一个仓库会包含同一个软件不同版本的镜像,而标签就常用于对应该软件的各个版本。我们可以通过
`<仓库名>:<标签>`的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签,将以`latest`作为默认标签。

以`Ubuntu`镜像 为例,`ubuntu`是仓库的名字,其内包含有不同的版本标签,如,`14.04`,`16.04`。我们可以通过`ubuntu:14.04`,或者`ubuntu:16.04`来具体指定所需哪个版本的镜像。如果忽略了标签,比如`ubuntu`,那将视为`ubuntu:latest`。

仓库名经常以两段式路径形式出现,比如`jwilder/nginx-proxy`,前者往往意味着`Docker Registry`多用户环境下的用户名,后者则往往是对应的软件名。但这并非绝对,取决于所使用的具体`Docker Registry`的软件或服务。


#### 3.3.1、 Docker Registry 公开服务

`Docker Registry`公开服务是开放给用户使用、允许用户管理镜像的`Registry`服务。一般这类
公开服务允许用户免费上传、下载公开的镜像,并可能提供收费服务供用户管理私有镜像。

最常使用的 `Registry`公开服务是官方的`Docker Hub`,这也是默认的`Registry`,并拥有大量的高质量的官方镜像。除此以外,还有`CoreOS`的`Quay.io`,`CoreOS`相关的镜像存储在这里;Google的`Google Container Registry`,`Kubernetes`的镜像使用的就是这个服务。

由于某些原因,在国内访问这些服务可能会比较慢。国内的一些云服务商提供了针对`Docker Hub`的镜像服务(`Registry Mirror`),这些镜像服务被称为加速器。常见的有阿里云加速器、`DaoCloud`加速器 等。使用加速器会直接从国内的地址下载`Docker Hub`的镜像,比直接从`Docker Hub`下载速度会提高很多。

国内也有一些云服务商提供类似于`Docker Hub`的公开服务。比如时速云镜像仓库、网易云镜像服务、`DaoCloud`像市场、阿里云镜像库 等。


#### 3.3.2、 私有 Docker Registry

除了使用公开服务外,用户还可以在本地搭建私有`Docker Registry`。`Docker`官方提供了`Docker Registry`镜像,可以直接使用做为私有`Registry`服务。

开源的`Docker Registry`镜像只提供了`Docker Registry API`的服务端实现,足以支持`docker`命令,不影响使用。但不包含图形界面,以及镜像维护、用户管理、访问控制等高级功能。在官方的商业化版本`Docker Trusted Registry`中,提供了这些高级功能。

除了官方的`Docker Registry`外,还有第三方软件实现了`Docker Registry API`,甚至提供了用
户界面以及一些高级功能。比如,`VMWare Harbor`和`Sonatype Nexus`。


## 四、 `Docker`的安装

`Docker`是一个开源的商业产品。`Docker CE`（社区版）安装参考官方文档。

[Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

**使用了国内源**
```
sudo apt-get update

sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common

curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add

 sudo add-apt-repository \
"deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
$(lsb_release -cs) \
stable"

sudo apt-get update
sudo apt-get install docker-ce

# 启动docker
sudo systemctl enable docker
sudo systemctl start docker

# 建立docker用户组
sudo groupadd docker
sudo usermod -aG docker $USER
```

安装完成后，需配置国内镜像加速,[参考这篇文章](https://github.com/yeasy/docker_practice/blob/master/install/mirror.md)：

>  在`/etc/docker/daemon.json`中写入如下内容（如果文件不存在请新建该文件）

    {
        "registry-mirrors": [
            "https://registry.docker-cn.com"
        ]
    }

之后重新启动docker服务：

    sudo systemctl daemon-reload
    sudo systemctl restart docker

检查加速器是否生效：

    sudo docker info
    # 看到如下内容说明配置成功 
    Registry Mirrors:
        https://registry.docker-cn.com/

**测试Docker是否安装正确**
```
docker run hello-world

# output
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete 
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
          ...........

```


## 五、 `Image`文件
**Docker把应用程序及其依赖，打包在 image 文件里面。**只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

image 是二进制文件。实际开发中，一个image文件往往通过继承另一个image文件，加上一些个性化设置而生成。举例来说，你可以在 Ubuntu 的 image 基础上，往里面加入 Apache 服务器，形成你的image。
```
# 列出本机所有的image文件
docker image ls

# 删除image文件
docker image rm [imageName]
```
**image 文件是通用的，一台机器的image文件拷贝到另一台机器，照样可以使用。**一般来说，为了节省时间，我们应该尽量使用别人制作好的image文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。

为了方便共享，image 文件制作完成后，可以上传到网上的仓库。Docker 的官方仓库`Docker Hub` 是最重要、最常用的 image 仓库。此外，出售自己制作的 image 文件也是可以的。

将image文件从仓库抓取到本地。

    docker image pull library/hello-world
    # docker image pull 是抓取image的命令。library/hello-world是image文件在仓库里面的位置，其中library是image文件所在的组，hello-world是image的名字。

由于Docker官方提供的image文件，都放在library组里面，所以它是默认组，可以省略，因此，上面的命令可以写成下面这样：

    docker image pull hello-world

抓取成功后，就可以在本机看到这个image文件了

    docker image ls

运行`image`文件命令如下：

    docker container run hello-world
    或者
    docker run hello-world

>  `docker container run`或者`docker run `命令会从image文件，生成一个正在运行的容器实例。如果本地没有找到image文件，docker就会从仓库自动抓取image文件，因此，前面的`docker image pull`命令不是必需的步骤。

<b style="color: red">注意:</b>有些容器不会自动终止，必须使用`docker container kill`命令手动终止。`docker container kill [containID]`


## 六、 容器文件
** `image`文件生成的容器实例，本身也是一个文件，称为容器文件。**也就是说，一旦容器生成，就会同时存在两个文件：image文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。

    # 列出本机正在运行的容器
    docker container ls
    #
    # 列出本机所有容器，包括终止运行的容器
    docker container ls -all
    #
    # 运行终止的容器文件，依然会占据硬盘空间，可以使用以下命令删除。
    docker container rm [containerID]

学会使用image文件以后，接下来的问题就是，如何可以生成image文件？如果你要推广自己的软件，势必要自己制作image 文件。这就需要用到`Dockerfile`文件。它是一个文本文件，用来配置image。Docker根据该文件生成二进制的 image 文件。下一章将通过一个实例来介绍如何编写`Dockfile`文件。



## 七、 实例：制作自己的Docker容器
下面我以[koa-demos](http://www.ruanyifeng.com/blog/2017/08/koa.html)项目为例，介绍怎么写`Dockerfile`文件，实现让用户在`Docker`容器里面运行`Koa`框架。
作为准备工作，请先[下载源码](https://github.com/ruanyf/koa-demos/archive/master.zip)，或者采用下面方式下载。

    git clone https://github.com/ruanyf/koa-demos.git
    cd koa-demos


### 7.1、 编写`Dockerfile`文件
首先，在项目的根目录下，新建一个文本文件`.dockerignore`,写入下面的内容，

```
.git
node_modules
npm-debug.log
```
上面的代码表示，这三个路径要排除，不要打包进入image文件，如果没有要排除的路径，可以不写。

然后，在项目的根目录下，新建一个文本文件Dockerfile。写入下面的内容。
```
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
```
上面五行代码，含义如下。

+ `FROM node:8.4`：该image文件继承官方的`node image`，冒号表示标签，这里标签是8.4，即8.4版本的 node。
+ COPY . /app：将当前目录下的所有文件（除了.dockerignore排除的路径），都拷贝进入 image 文件的/app目录。
+ WORKDIR /app：指定接下来的工作路径为/app。
+ RUN npm install：在/app目录下，运行npm install命令安装依赖。注意，安装后所有的依赖，都将打包进入 image 文件。
+ EXPOSE 3000：将容器 3000 端口暴露出来， 允许外部连接这个端口。


### 7.2、 创建image文件
有了Dockerfile文件以后，就可以使用`docker image build`命令创建image文件了。

    docker image build -t koa-demo .
    # 或者
    docker image build -t koa-demo:0.0.1 .

在上面的代码中，`-t`参数用来指定image文件的名字，后面还可以用冒号指定标签。如果不指定，默认的标签就是`latet`。最后的那个点`.`表示`dockfile`文件所在的路径，上例是当前路径，所以是一个点。如果运行成功就可以看到新生成的image文件`koa-demo`了。

    # 查看image文件 
    docker image ls


### 7.3、 生成容器
`docker container run`命令会从`image`文件生成容器。
```
docker container run -p 8000:3000 -it koa-demo /bin/bash
# 或者
docker container run -p 8000:3000 -it kao-demo:0.0.1 /bin/bash 
```
上面命令的参数含义如下：

+ -p参数：容器的3000端口映射到本机的8000端口
+ -it参数：容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器。
+ koa-demo:0.0.1：image 文件的名字（如果有标签，还需要提供标签，默认是 latest 标签）。
+ /bin/bash：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。

如果一切正常，运行上面的命令后，就会返回一个命令行提示符。

    root@805724fbb49b:/app# 

这就表示你已经在容器里面了，返回的提示符就是容器内部的`Shell`提示符。执行下面的命令。

    root@805724fbb49b:/app# node demos/01.js

这时，Koa 框架已经运行起来了。打开本机的浏览器，访问`http://127.0.0.1:8000`，网页显示"Not Found"，这是因为这个 demo 没有写路由。

这个例子中，Node 进程运行在 Docker 容器的虚拟环境里面，进程接触到的文件系统和网络接口都是虚拟的，与本机的文件系统和网络接口是隔离的，因此需要定义容器与物理机的端口映射（map）。

现在，在容器的命令行，按下 Ctrl + c 停止 Node 进程，然后按下 Ctrl + d （或者输入 exit）退出容器。此外，也可以用`docker container kill`终止容器运行。

    #在本机的另一个终端窗口，查处容器的ID
    docker container ls
    #
    # 停止指定的容器运行
    docker container kill [containerID]

容器停止运行后不会消失，用下面的命令删除容器文件。
```
# 查处容器的ID
docker container ls -all

# 删除指定容器文件
docker container rm [containerID]
```
>  也可以使用`docker container run`命令的`--rm`参数，在容器终止运行后自动删除容器文件。
>  `docker container run --rm -p 8000:3000 -it koa-demo /bin/bash`


### 7.4、 `CMD`命令
上一节的例子里面，容器启动以后，需要手动输入命令`node demos/01.js`。我们可以把这个命令写在`Dockerfile`里面，这样容器启动以后，这个命令就已经执行了，不用再手动输入了。
```
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
CMD node demos/01.js
```
上面的 Dockerfile 里面，多了最后一行`CMD node demos/01.js`，它表示容器启动后自动执行`node demos/01.js`。

你可能会问，`RUN`命令与`CMD`命令的区别在哪里？简单说，`RUN`命令在`image`文件的构建阶段执行，执行结果都会打包进入`image`文件；`CMD`命令则是在容器启动后执行。另外，一个`Dockerfile`可以包含多个`RUN`命令，但是只能有一个`CMD`命令。

注意，指定了`CMD`命令以后，`docker container run`命令就不能附加命令了（比如前面的`/bin/bash`），否则它会覆盖`CMD`命令。现在，启动容器可以使用下面的命令。

    docker container run --rm -p 8000:3000 -it koa-demo

### 7.5、 发布`image`文件
容器运行成功后，就确认了 image 文件的有效性。这时，我们就可以考虑把 image 文件分享到网上，让其他人使用。

首先，去 hub.docker.com 或 cloud.docker.com 注册一个账户。然后，用下面的命令登录。

    docker login

然后，为本地的image标注用户名和版本。
```
docker image tag [imageName] [username]/[repository]:[tag]
# 实例
docker image tag koa-demos:0.0.1 heany/koa-demos:0.0.1
```

也可以不用标注用户名，重新构建一下image文件。

    docker image build -t [username]/[repository]:[tag]

最后，发布image文件。

    docker image push [username]/[repository]:[tag]

发布成功后，登录hub.docker.com，就可以看到已经发布的image文件。<b style="color: red">目前国内链接`hub.docker.com`非常慢，push经常超时，可以使用国内的`daocloud`。</b>


## 八、 其他有用的命令
`docker`的主要用法就是上面这些此外还有几个命令，也非常有用。

### 8.1、 `docker container start`
前面的`docker container run`命令是新建容器，每运行一次，就会新建一个容器。同样的命令运行两次，就会生成两个一模一样的容器文件。如果希望重复使用容器，就要使用`docker container start`命令，它用来启动已经生成、已经停止运行的容器文件。

    docker container start [containerID]

### 8.2、 `docker container stop`
前面的`docker container kill`命令终止容器运行，相当于向容器里面的主进程发出`SIGKILL` 信号。而`docker container stop`命令也是用来终止容器运行，相当于向容器里面的主进程发出`SIGTERM`信号，然后过一段时间再发出 SIGKILL 信号。

    bash container stop [containerID]
这两个信号的差别是，应用程序收到`SIGTERM`信号以后，可以自行进行收尾清理工作，但也可以不理会这个信号。如果收到`SIGKILL`信号，就会强行立即终止，那些正在进行中的操作会全部丢失。


### 8.3、 `docker container logs`
`docker container logs`命令用来查看`docker`容器的输出，即容器里面`Shell`的标准输出。如果`docker run`命令运行容器的时候，没有使用-it参数，就要用这个命令查看输出。

    docker container logs [containerID]


### 8.4、 `docker container exec`
`docker container exec`命令用于进入一个正在运行的`docker`容器。如果`docker run`命令运行容器的时候，没有使用-it参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的`Shell`执行命令了。

    docker container exec -it [containerID] /bin/bash

### 8.5、 `docker container cp`
`docker container cp`命令用于从正在运行的`Docker`容器里面，将文件拷贝到本机。下面是拷贝到当前目录的写法。

    docker container cp [containID]:[/path/to/file]


## 九、 结束语
&emsp;转载于[阮一峰的网络日志-Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
&emsp;参考了[Docker技术入门与实战](https://yeasy.gitbooks.io/docker_practice/content/)