---
title: Docker学习笔记
layout: post
category: learning-notes
tags: [ Docker, learning-notes ]
comments: true
---

这周末有一个庆祝Docker四周岁的Party，被学长领着参加了。为了避免去那边之后dalao的话一句都听不懂只能吃蛋糕，花了一晚上恶补了Docker的一些基础知识，这里稍作记录。

<!-- more -->

## 安装

安装的教程没什么好贴的，这个也不是从源码编译，说多了没意思。

本机用的是Ubuntu，所以就列个Ubuntu的安装吧。为了快速学习起见，（难得）直接用`apt`下载来了。Ubuntu的命令是`apt install docker.io`可以直接安装Ubuntu源的docker，直接install docker不能用，不是很懂。

Arch Linux直接`pacman -S docker`就好了。

安装完毕后，`docker run hello-world`试试看能不能运行，hello-world是官方提供的测试镜像，注意要以root权限运行。

## 配置

如果试图直接跑hello-world就会发现首先docker提示本地没有这个镜像Unable to find image 'hello-world:latest' locally，docker会很贴心地自己去服务器下载。然后再等一会儿，报错……（看了一眼报错才发现是Go写的，net/http不就是Go的一个库么）

好了，最近领导们开个会我也不好乱说话，总之为了能成功下载到这个镜像，需要去更改docker的源。

从<a href="https://lug.ustc.edu.cn/wiki/mirrors/help/docker" target="_blank">USTC的LUG</a>查来的资料（大爱USTC），修改（没有就新建）/etc/docker/daemon.json即可。

{% highlight json %}
{
    "registry-mirrors": ["https://here.is.your.mirros"]
}
{% endhighlight %}

然后再试试`docker run hello-world`就好了，打印出一堆欢迎信息。

## 概念

没有仔细去研究过这些名词，如果有错误还请指出。

我是这么理解docker的，docker本身就像是个虚拟机，提供虚拟化服务，可以在系统上跑另一个系统，至于Linux能不能运行Windows，理论上不能，实际上，我们进一个容器里查看内核版本的时候，会发现其实和宿主机是一样的，也就是本质上docker内的系统还是跑在宿主机系统内核上的。但是实际上参见<a href="http://blog.daocloud.io/dockercon-day-2-jessie-image/" target="_blank">这篇文章</a>，虽然并不是真正意义上的跑Windows。

尽管知道Docker并不是真的虚拟机，我就假装是了，轻喷。如果像我一样不是想研究原理，当作虚拟机用就足够了。

docker里的镜像，也很类似于iso镜像文件，是只读的，是用来创建容器的。而至于容器，可以类比成真的被创建出来的虚拟机，我们可以进去执行任意操作，并且也会被保存下来，也就是不仅可读也可写。

每个容器都会被赋予一串ID，但是ID是随机的16进制字符串，难记，同时也会被分配一个随机的name，一般来说是形容词-下划线-名人名字的形式，也可以用来指明一个容器的身份。当然，name也可以自己设置。

## 几个简单的命令

`docker run`，新建一个容器，运行一个镜像。

几个比较常用的option有`-d`，也就是`--detach`，即分离模式，后台运行。`-i`也就是`--interactive`，交互模式运行，打开STDIN，默认是false。`--name`可以设置新建的容器的name。`-P`和`-p`，分别是`--publish-all`和`--publish`，前者将所有端口映射到宿主机的端口上，后者自定哪些端口映射到宿主机的哪些端口。以及`-t`，`--tty`，为虚拟机在宿主机上新建一个tty作为STDOUT。

举例而言，我们想跑一个Arch Linux，可以直接run，由docker从hub下载镜像。

{% highlight bash %}
docker run -it base/archlinux /usr/bin/bash
{% endhighlight %}

上面的命令的意思就是交互模式运行base/archlinux里的/usr/bin/bash，如果本地没有base/archlinux，docker的守护进程会直接从hub里下载镜像，运行的时候会新建一个容器。

更一般地说，docker run的套路是这样的：`docker run [OPTIONS] IMAGE [COMMAND]`

如果不想通过run来下载，可以直接使用`docker pull`下载。另外，如果自己在一个hub里有帐号什么的、类似github，可以通过`docker push`来把自己制作的镜像push上去。至于怎么制作镜像，后面会讲。

想要退出容器，可以直接用`exit`，`logout`之类的命令，也可以<kbd>Ctrl</kbd><kbd>D</kbd>退出，但是这样相当于直接关机，下次启动容器需要用`docker start`再加上`docker attach`才行。如果希望在后台运行，可以用<kbd>Ctrl</kbd><kbd>P</kbd>或者<kbd>Ctrl</kbd><kbd>Q</kbd>来detach，到时候直接`docker attach`就可以进入。

退出了这个容器之后，可以通过`docker ps -a`来检查所有的容器。通过`docker images`来检查所有的镜像。如果想要查看一个detach的容器的输出，可以通过`docker logs CONTAINER`来查看。

如果想要重启一个容器，使用`docker start CONTAINER`启动，通过`docker attach CONTAINER`连接进去。或者直接用`docker exec CONTAINER COMMAND`执行单个命令，和`docker run`差不多。

不用了，可以删除容器和镜像。可以用`docker rm CONTAINER`来删除容器，通过`docker rmi IMAGE`来删除镜像。如果一个镜像已经创建了容器，是不能直接删除的，要不加上`-f`也就是`--force`参数，要不通过删除所有的容器来进行。

具体的选项都不解释了，可以从`man`来看。

## 创建一个镜像

没有这么难，如果想要自己创建一个镜像，我们可以先从别人的镜像开始，进去编辑编辑，退出来保存成镜像。直接用`docker commit`就可以极快地通过一个容器创建一个镜像。

比较常用的几个选项是`-a`即`--author`，一般填自己姓名和邮箱。`-m`即`--message`，commit的一些信息描述。还有一个`-c`和`--change`，是具体说明一些Dockerfile的内容的。

### Dockerfile语法简介

这里具体要讲的是从Dockerfile创建一个镜像。语法非常简单，关键词很少，主要还是需要会写shell script，这一点和Makefile很像。

Dockerfile的语法就只有一种，`INSTRUCTOR argument`，INSTRUCTOR是一些操作，不区分大小写，不过约定俗成都是大写。argument则是一些命令。

首先新建目录，在其下新建一个Dockerfile文件，文件名可以为Dockerfile和dockerfile。在这里，我们首先来写一个简单的hello-world，运行镜像只输出一行“Hello, World!”。

Dockerfile的第一句必须是`FROM`，`FROM`后面接的镜像是你写的镜像在什么镜像基础上建立。比如`FROM ubuntu`，就是在ubuntu的镜像的基础上建立你自己的镜像。

如果想要从零开始新建一个镜像呢？也可以，docker有一个保留字叫做`scratch`，`FROM scratch`就是从零开始新建项目。所谓从零开始，就是里面真的什么都没有，只有一个根目录。

一般而言，第二个命令我们都会写`MAINTAINER`，后面写一些自己的信息，姓名、邮箱之类的。

然后我们可以用`ADD`命令给镜像里添加一些宿主机的文件，语法是`ADD src dst`，src是文件在宿主机里的地址，`dst`是添加到宿主机的哪里。比较有趣的是src可以是一个URL。

设置工作目录用`WORKDIR`，后面直接接目录。

设置环境变量使用`ENV`，语法是`ENV key alue`。

设置用户使用`USER`，后面接uid就好了。

想要执行一些命令，用`RUN cmd`来执行，`cmd`一般都是shell脚本。

想要映射端口，使用`EXPOSE`命令，使用`EXPOSE 80`可以打开一个容器的端口，`EXPOSE 80:8080`的意思是将容器内的80端口映射到主机的8080端口。一般不建议在Dockerfile里映射端口，因为可能会导致冲突而无法启动，而且用户也可以直接在执行时映射端口，没必要在创建时设置。

`VOLUME`可以让容器内访问到宿主机的文件系统，语法是`VOLUME ["directory1", "directory2"]`。

最后，使用`ENTRYPOINT`来设置容器的启动命令，语法为`ENTRYPOINT ["executable", "param1", "param2"]`，可以不加方括号，但是建议用前者。这样的话，直接运行`docker run`运行这个容器，后面接参数就相当于直接执行了这个executable文件。

`CMD`有三种形式`CMD ["executable", "param1", "param2"]`就是普通地执行一个可执行文件并为其提供一些参数。`CMD ["param1, "param2"]`则是为`ENTRYPOINT`提供参数。还有一种不加方括号的`CMD cmd param1 param2`等价于执行`/bin/sh -c 'cmd param1 param2'`，一般使用前两者。

需要注意的是，`ENTRYPOINT`和`CMD`均只能出现一次，如果出现多次，则只会执行最后的那句。

### 从Dockerfile新建

既然都没问题，可以简单地写一个Dockerfile了。我打算从零开始，也就是使用`FROM scratch`来做我的镜像。

因为scratch里什么都没有，我们直接扔一个可执行文件进去的后果就是动态链接的库全部都找不到，那这个程序就运行不了，因此，我们就需要编译成静态链接的可执行文件。

简单写一个hello-world.c，如下

{% highlight c %}
int puts(const char s[]);
int main() {
    puts("Hello, World!");
    return 0;
}
{% endhighlight %}

静态链接`gcc -static -o hello-world hello-world.c`，检查一下是否有动态链接，`ldd hello-world`，应该会告诉我们不是动态可执行文件，这就好了。

然后Dockerfile不是很难。

{% highlight dockerfile %}
FROM scratch
MAINTAINER sir-Xris

ADD hello-world /
ENTRYPOINT /hello-world
{% endhighlight %}

就是这么简单。直接在当前目录下运行`docker build --tag hello-world .`就做好了一个镜像。来跑跑看？`docker run hello-world`应该是直接打印出一行Hello, World!。

如果需要更复杂的，不必非要从scratch开始建造，换个其它的也可以。

更多的build选项可以查看mannual。
