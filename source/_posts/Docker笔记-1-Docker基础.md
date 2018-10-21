---
title: 'Docker笔记(1):Docker基础'
date: 2018-10-21 17:15:24
tags:
	-- docker
---

近年来云计算技术发展十分迅速，其中容器化技术是云计算未来发展的重要趋势之一，而`Docker`是目前最为成熟和便捷的容器技术。`Docker`是`Moby（原 Docker Inc.）`基于`Golang`开发的容器化技术，底层基于`LXC(Linux Container技术，需内核支持)`。`Docker`至今已经得到众多云计算厂商一致认可和主动支持，很多大厂已经开始自研新的容器化技术。本系列主要是记录笔者在研究学习`Docker`技术以及相关技术栈的心得体会，本篇首先从基础概念讲起。

<!--more-->

### 安装 Docker

`Docker`支持运行在`MacOS`,`Linux`和`Windows`平台上运行，分为`CE(开源社区版)`和`EE(商业版本)`两种版本。

社区版本缺少一些管理功能，但完全可以满足需求。不同平台的安装流程不同，官方文档有详细的说明。这里给出`Ubuntu`和`MacOS`的安装链接：

Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/#upgrade-docker-ce
MacOS: https://docs.docker.com/docker-for-mac/install/

需要注意的是, 有些旧版本有不兼容的更新，升级需注意; 
如果是安装最新版本则不会遇到兼容问题;
更多说明参见 Faq: https://docs.docker.com/engine/faq/

安装完成后可以通过命令确认:
```bash
~ docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 ... # 省略

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  ...  # 省略
```

### Docker 镜像与容器

通俗地说，`Docker`是把项目程序和运行环境进行打包(文件)的技术,然后就可以便捷地分发到新的主机上快速部署和运行程序。

打包得到的文件就是`镜像(文件)`;而读入`镜像`文件，运行起来的程序可以称之为`容器`（项目的程序进程在隔离的`容器`环境中运行，所以可以理解为`容器`是在`镜像`在内存中运行起来的形态）。
这里是官方的解释: https://docs.docker.com/get-started/#docker-concepts

虚拟机也算虚拟化技术的一种，那么容器和虚拟机有什么区别?经常听到容器技术比虚拟机更高效，这又是为什么?下面的对比图可以很清晰的解释其中的原因:

{% asset_img container_vs_vm.png local file %}

linux上运行的docker容器是众多容器共享系统内核,在容器中独立运行程序进程，进程级别做抽象;而虚拟机是通过`Hypervisor`抽象层在宿主机上层运行多个操作系统，然后在操作系统内部运行程序来实现隔离，系统级别抽象；就运行效率来说明显是docker优胜，但是这种共享内核的方式也可能会带来安全问题。总之这两种技术各有长短，选型还是要看场景，而且某些场景下也有二者结合使用的，这里暂且按下不表;


下面来执行一条命令：

```bash
~ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
...  # 省略
```

上述过程是:首先在本地没有找到`hello-world`镜像;从远程的`镜像仓库(后文解释)`下载镜像；
然后生成相关容器执行程序(一般容器都会配置一个默认命令), 程序执行完毕后退出进程结束，然后容器也退出了;

```bash
docker images -a  # 列出所有镜像
docker container ls -a  # 列出所有的镜像
docker pull <image>  # 拉取镜像
docker run <image>  # 运行镜像(默认会创建新镜像)
docker rm <name/container_id>  # 删除容器
docker rmi <image>  # 删除镜像(只有先删除干净容器才可以删除镜像)
```

镜像运行结束后生成的镜像并没有被删除(`docker container ls -a`可以列出我们运营过的`hello-world`镜像, 而且一个镜像可以生成多个容器);在容器中做的读写也不会消失，除非把容器删除；

```bash
docker pull ubuntu  # 拉去远程的ubuntu镜像
docker run -it --rm ubuntu bash  # ubuntu容器中运行shell;-it,提供一个交互式会话shell;--rm,结束后删除容器
docker run -d --rm -p 80:80 nginx  # 运行一个nginx容器, nginx -g daemon off 默认在后台持续运行; 80端口映射外部
docker logs <name/container_id>  # 展示容器log, 标准输出
docker stats <name/container_id>  # 展示容器中进程状态
docker exec <name/container_id> <cmd> # 在容器中执行命令
docker inspect <name/container_id> # 展示容器原始数据
```

docker支持很多对镜像和容器管理的命令，这些命令都还支持很多可选参数;更具体的内容请参看手册;

到这里，我们知道了镜像和容器的概念，以及我们可以方便地分发镜像和运行容器中的程序;
其实，容器最大的好处就是由同一个镜像生成的所有容器的环境都是一致的，这样我们就可以轻松实现开发和部署的环境统一!

### 创建镜像与Dockerfile

看起来容器使用起来缺失很方便，但是我们如何用自己的项目构建镜像？
最常用的镜像构建方式有两种: 1. 使用`Dockerfile`创建; 2. 从容器构建;

* 使用`Dockerfile`构建镜像

1. 新建一个空文件夹(一般可以是项目目录)，并在其中新建一个`Dockerfile`文件

`Dockerfile`文件内容如下:
{% gist 009a15d3368ab58b7e310392a0485018 %}

2. 使用`docker build -t demo .`(注意不要少了结尾的`.`)
3. 使用`docker image`就可以看到新创建的有`demo`标签的镜像

有关指令的具体解释参见文档: https://docs.docker.com/engine/reference/builder/#usage

通常都用这种方式来从一个基础镜像上构建项目的镜像；
`Dockerfile`因尽量减少语句条数，提高中间镜像的重复使用率;
因为每条指令会在原有的镜像上新加一层生成新的镜像; 所有的指令都执行完毕后就得到了结果镜像;
这跟docker的原理息息相关，这里暂且不表;

* 从现有容器构建

首先回想下前面说过，镜像是打包好的文件，而镜像是运行在内存里的程序；
事实上我们的镜像是一层一层构建起来的，就像我们在`Dockerfile`中那样逐层构建镜像；而容器可以理解为只读的镜像层的最上层再加一层可读写层；
对一个容器我们可以使用`docker commit`从容器构建新的镜像，实际就是把当前容器的读写层当如到新的镜像中;

```bash
~ docker commit 73deeb84a482 first_image
```

### 镜像仓库

镜像仓库是`Docker`技术栈中的重要组件，主要用于存储和分发，各个版本或标签的多个镜像;
可以接收用户推送(`docker push`)来的镜像进行存储,也可以把镜像分发出去(`docker pull`)；
例如：某景象名称`distribution/registry`, 被打标签`2.0`和`latest`(一个镜像支持多标签)；

默认的镜像仓库是: https://index.docker.io/v1/, 使用`docker info`可以查看;
如果想把自己的镜像推送到`docker hub`仓库中首先需要注册账号, 地址: https://hub.docker.com;

```bash
~ docker login
...
Username: yabinzhang
Password:
Login Succeeded  # 登录成功

~ docker tag first_image docker.io/yabinzhang/first_image  # 打标签
sha256:ff08bff7a30df3da56f561712ba7fee4d11020fae7656f571712fb907f5fb9ea
~ docker push docker.io/yabinzhang/first_image
...
```
登录http://hub.docker.io, 就可以看到自己推送的镜像了;
使用`docker pull yabinzhang/first_image`就可以下载到刚推送的镜像了;
官方打包了一个镜像便于我们很快地部署私有的仓库，具体内容参见 https://hub.docker.com/_/registry/

### 总结

总结一下:

1. 本篇介绍了镜像和容器的基本概念, 容器是"运行的镜像"，镜像是"持久化的容器";
2. 介绍了容器和镜像的管理操作和基本使用;
3. 介绍了`Dockerfile`和`docker commit`两种创建镜像方法;
4. 介绍了镜像仓库和镜像的推送与分发；

通过本篇docker基础的学习，我们已经具备了把简单的小项目容器化，然后分发和部署的能力了；
后续我们将学习在多容器化服务与服务编排。