---
layout: post
title: Arch Linux安装小记
description: Arch Linux安装小记
category: configuration
tags: [ Linux, 运维 ]
comments: true
---

因为在招新期间比较蛋疼地装了虚拟机然却删了详细记录安装历程的Write Up，而另一份Write Up太过粗糙简陋，没有说明那些引导文件如何编辑，以至于连我自己都无法按照那些步骤再次安装，于是就有了这次重装经历。

<!-- more -->

另外这个系统打算私用，不会去考虑兼容性。

## 配置虚拟机

虚拟机用的是VMware® Workstation 12 Pro，不考虑兼容性，所以使用的是Workstatin 12.0的硬件兼容性，Arch Linux版本是archlinux-2016.11.01-dual.iso，客户机操作系统里选择Linux的其它Linux 3.x内核 64位（实际上Arch Linux的内核已经是最新的Linux 4.8.1了来着），注意不选择64位的话将会导致UEFI模式下引导光盘失败。

虚拟机硬件配置内存4GB，处理器一个双核，8GB硬盘。使用UEFI引导，所以下文要编辑引导文件，生命在于折腾。使用NAT网络地址转换，暂时不想我的虚拟机在局域网公开，之后可能会换成桥接用手机ssh。其它参数不重要。

VMware Workstaion默认不开启UEFT，至于怎么使用UEFI，<a href="https://communities.vmware.com/docs/DOC-28494" target="_blank">这篇文章</a>有很详尽的介绍

## 安装过程

开机即可进入光盘引导，直接第一项回车即可。

熟悉的界面。

![]({{ site.url }}{{ site.baseurl }}img/archlinux-begin.png)

其实Wiki说得很清楚，Arch的安装主要是要耐下心来去找资料。首先一开始的系统配置都很简单，跟着套路一步一步走即可。

### 各项检查及系统配置

呃虽然VMware的UEFI检验过可行，不过还是按照套路来吧。简单ls一下即可。

{% highlight bash linenos %}
ls /sys/firmware/efi
{% endhighlight %}

如果正常输出则为EFI模式启动的。

然后如果要设置键盘布局，可以用`loadkeys`更改，所有可用的键盘布局可以用`ls /usr/share/kbd/keymaps/**/*.map.gz`来显示，默认一般是qwerty的us布局。

检查一下联网

{% highlight bash linenos %}
ping -c 4 xris.co
{% endhighlight %}

顺便更新时间

{% highlight bash linenos %}
timedatectl set-ntp true
{% endhighlight %}

虽然不知道为什么我这里时间非常奇特，更新不了。

### 硬盘分区和格式化

毫无疑问分区是必须的。因为用的是UEFI，所以用gdisk分一下GPT的区。

{% highlight bash linenos %}
gdisk /dev/sda
> n             # 新建分区
> 1             # 设为一号分区
> 2048          # 默认起始扇区
> 1050623       # 1048576个扇区，512MB
> ef00          # 设置为EFI分区
> n             # 新建分区
> 2             # 设为二号分区
> 1050624       # 默认起始扇区
> 16777182      # 默认终止扇区
> 8300          # 默认分区类型
> w             # 写磁盘并退出
> Y             # 确认进行操作
{% endhighlight %}

稍等一下，分区完毕。`ls`一下/dev可以看见两个/dev/sda1和/dev/sda2。

我把/dev/sda1留作UEFI的启动分区，/dev/sda2作为根分区，于是乎按照要求，我给/dev/sda1以fat32格式，/dev/sda2以ext4格式。

{% highlight bash linenos %}
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
{% endhighlight %}

分区完毕后就可以挂载、下载软件，然后切换root了。

### 安装系统

首先挂载一下

{% highlight bash linenos %}
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
{% endhighlight %}

然后直接下载就安装好了

{% highlight bash linenos %}
pacstrap /mnt base base-devel
{% endhighlight %}

生成一下fstab文件

{% highlight bash linenos %}
genfstab -U /mnt >> /mnt/etc/fstab
{% endhighlight %}

切换到硬盘系统中

{% highlight bash linenos %}
arch-chroot /mnt
{% endhighlight %}

然后就是设置时区、语言环境、主机名之类

{% highlight bash linenos %}
### 设置时区
ln -s /usr/share/timezone/Asia/Shanghai /etc/localtime
hwclock --systohc --localtime
### 将/etc/locale.gen中的需要的取消注释即可
vim /etc/locale.gen
locale-gen
### 初始化内存
mkinitcpio -p linux
### 开机自启动网络
systemctl enable dhcpcd
{% endhighlight %}

### 启动引导

我选择<a href="https://wiki.archlinux.org/index.php/Systemd-boot" target="_blank">systemd-boot</a>，用`bootctl`来配置启动项。

几个用得到的sample文件都放在`/usr/share/systemd/bootctl`目录下，感觉人生失去希望可以去看一下。

{% highlight bash linenos %}
mount /dev/sda1 /mnt
bootctl --path=/mnt install
{% endhighlight %}

编辑`/mnt/loader/loader.conf`如下

{% highlight bash %}
default arch
timeout 0
editor  0
{% endhighlight %}

编辑`/mnt/loader/entries/arch.conf`如下，其中PARTUUID由命令`blkid -s PARTUUID -o value /dev/sda2`获取

{% highlight bash %}
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=3a60417f-3195-408d-be1f-320c9b850704 rw
{% endhighlight %}

然后最后退出，重启即可。

![]({{ site.url }}{{ site.baseurl }}img/welcome-to-archlinux.png)
