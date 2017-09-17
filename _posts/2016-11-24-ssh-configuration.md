---
layout: post
title: ssh简要介绍与配置
category: configuration
description: ssh一些介绍和安装配置到使用的过程
tags: [ ssh, Linux, 密码学 ]
comments: true
---

帮舍友折腾了一会儿ssh的安装以及各项配置，突然想起来自己工作室招新时写的ssh配置的东西还没发上来，于是打算乘此机会稍微修改一下补充一下内容……然后回头看了眼Archlinux安装记录，果然是小记只记了步骤，根本写“为什么这么做”及其它……

ssh的配置其实就下载一下，然后取消掉密码登录的事儿，实际上也感觉没什么好写，所以会稍微扯扯……一些相关资料

<!-- more -->

## 关于SSH

SSH全称是Secure Shell，是被开发出来用于安全的数据交换，可以用于取代传输不加密的Telnet，通常用于远程访问和执行命令，甚至可以做到X11回话的转发。因为原来的SSH是一个有版权的商业产品，于是某日在一个openBSD版本里开发了开源的openSSH这玩意儿用来替代SSH，于是……因为开源的通常比闭源的可靠+值得信任，于是大家就都用开源的了。所以下文中提到的ssh均指代openSSH。

开源抄了闭源……这让我突然想到一句话：

> 开源软件对使用GPL代码却不开源的人的最有力还击就是把它的功能抄过来再开源了。 ——clowwindy

## 安装配置ssh

在绝大部分的发行版上都已经配置安装好了ssh和sshd，亦即客户端和服务器端均配置完毕。特别是Server级别的Linux产品，sshd是装机必备。但是如果没有ssh和sshd，我们通常可以一个命令直接下载安装配置。

常见的有两种包管理系

### Debian系

{% highlight bash %}
apt-get install ssh
{% endhighlight %}

### RHEL系

{% highlight bash %}
yum -y install openssh-server openssh-clients
{% endhighlight %}


### Archlinux

{% highlight bash %}
pacman -S openssh
{% endhighlight %}

好了，恭喜，可以使用ssh连接远程服务器以及被连接了。方法很简单

{% highlight bash %}
ssh $REMOTE_USER@$REMOTE_SERVER
{% endhighlight %}

~~全剧终~~，才怪

## 中间人攻击

这里简要介绍一下著名的中间人攻击(Man-in-the-middle Attack)。

简单来说，中间人攻击就是在通讯刚建立的时候，一个窃听者通过拦截服务端和客户端的消息并转发以获取相关信息。在一次攻击中，中间人对于服务端扮演的是客户端的角色，对于客户端扮演的是服务端的角色，通过拦截消息，将公私钥替换为自己的公私钥，就可以无障碍解读消息并且转发。

ssh在第一次连接新的服务器的时候并没有什么比较好的方法确实是否为真实主机，只有通过服务端指纹识别，人工比对来确认连接是否正确，此时ssh会给出以个警告并询问是否信任。

{% highlight bash %}
The authenticity of host 'cccc.cc (ddd.ddd.ddd.ddd)' can't be established.
ECDSA key fingerprint is xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx.
Are you sure you want to continue connecting (yes/no)
{% endhighlight %}

信任之后，主机信息就会被加入到~/.ssh/known\_hosts中。每次连接ssh都会确认信息，避免中间人的插足。

## 使用RSA加密登录

### 关于

所谓公钥，就是公开的(public)密钥，而相对的，私钥就是私人的(private)密钥，不能泄露的。公私钥相匹配，一般有两个用途：

* 私钥加密，公钥解密，一般用于数字签名。
* 公钥加密，私钥解密，一般用于加密传输。

具体可以去搜相关资料。在ssh中，两种方法都有使用，就是典型的使用方法。

**用户密码认证**是公钥加密。原理简单粗暴，客户端申请连接后，服务端返回其公钥，客户端在本地用公钥加密了用户输入的密码之后传回服务端，若服务端私钥解密后结果和真实的匹配即认证成功。

**RSA加密认证**是私钥加密。首先用户必须在在客户端本地生成一个公钥和相匹配的私钥，通过各种途径告诉服务端我们的公钥以后，每次发起认证申请，服务器在已认证公钥(一般保存于`~/.ssh/authorized_keys`)中找到客户端，直接返回一串随机字符，客户端用私钥加密发回，服务端用公钥解密后确认信息。

以上，之所以说到公私钥，原因是暴露在公网上的服务器在遭受着各种各样的攻击，比如密码爆破。第一种方法我们可以猜得出，密码是唯一的认证凭证，只要有了密码就可以登录到服务器并为所欲为。因此通常来说，放置于公网的服务器每天起来能看到尝试登录的次数上千甚至上万。

### 生成

所以为了避免服务器密码被猜中、暴力跑出，ssh也可以采取私钥加密的数字签名认证，这也是我说这么多的意义。

首先使用ssh自带的`ssh-keygen`来生成密钥

{% highlight bash %}
ssh-keygen -t ecdsa -b 521
{% endhighlight %}

接下来会有两步生成密钥文件和请求密码的确认提示，**密码可以留空**，但是建议不留空，防止被拿到私钥就被登上服务器，在最后再设置另ssh保存密码。

其中`-t`表明加密类型，可能的加密类型有“rsa1”、“dsa”、“ecdsa”、“ed25519”和“rsa”，ssh从5.3版本开始推荐使用ecdsa加密，密钥短而且加密强。

然后`-b`表明加密位数，这个和加密类型有关，举例而言我的ecdsa是521位的。

另外还有几个可能比较常用的比如`-q`是静默生成，`-N`是设置新密码，`-f`是自定义密钥文件，具体还是去`man ssh-keygen`看吧。简单点只要确认生成密码算法就可以了，或者确认一下长度。其它的都会在后面提示。

生成完毕后，会出来一个指纹和一张诡异的随机图片，我们可以无视这些……生成的密钥文件，如果没有特别指定，都会保存在~/.ssh/下。检索~/.ssh/会发现生成了两个文件，举ecdsa为例，会有id\_ecdsa和id\_ecdsa.pub，其中带有.pub的为公钥，需要传输至服务器。另一个是私钥，需要好好保管。

### 传输公钥

我们可以使用

{% highlight bash %}
ssh-copy-id -i ~/.ssh/id_ecdsa.pub REMOTE_USR@REMOTE_HOST
{% endhighlight %}

直接拷贝，或者手动拷贝，将~/.ssh/id\_ecdsa.pub手动`scp`到服务器上之后，添加到服务器~/.ssh/authorized\_keys后面即可。

{% highlight bash %}
# 以下是localhost上的操作
scp ~/.ssh/id_ecdsa.pub xris@remotehost:
ssh xris@remotehost
# 以下是remotehost上的操作
cat ~/id_ecdsa.pub >> ~/.ssh/authorized_keys
rm -v ~/.id_ecdsa.pub
{% endhighlight %}

## 服务端禁用密码登录

sshd服务的配置均在/etc/ssh/sshd\_config中，我们找到三行，注意三行是不一定连续的……我只是打在一起……

{% highlight bash %}
RSAAuthentication       yes
PubkeyAuthentication    yes
PasswordAuthentication  yes
{% endhighlight %}

确保`RSAAuthentication`和`PubkeyAuthentication`值为`yes`，以及，把`PasswordAuthentication`值改为`no`。

接下来让sshd重新加载配置文件。

{% highlight bash %}
systemctl reload sshd
{% endhighlight %}

如果有需求，禁止以root身份登录，那么在/etc/ssh/sshd\_config中找到

{% highlight bash %}
PermitRootLogin without-password
{% endhighlight %}

改成`no`。

以上，配置完毕。

## 免去私钥密码输入

因为在生成密钥的时候可以选择空密码，所以这一步设置了空密码的可以略去。

我们使用`ssh-agent`来保存我们用`ssh-add`输入的密码，首先运行`ssh-agent`

{% highlight bash %}
eval `ssh-agent`
{% endhighlight %}

出来一个Agent pid xxx，可以`ps -fp $SSH_AGENT_PID`来确认ssh-agent的运行状态。

接下去简单地

{% highlight bash %}
ssh-add
{% endhighlight %}

输入一下密钥就可以了，之后只要ssh-agent进程不断，直接远程到远程服务器不需要再输入密码，而且私钥直接被盗走也不一定会被猜到密码。
