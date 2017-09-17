---
title: Linux下的权限管理
layout: post
category: linux
tag: [ linux, privilege ]
comments: true
---

在WeChall刷题的时候遇到了几题Linux系统的题目，被[其中一题](http://www.wechall.net/en/challenge/warchall/choose_your_path2/index.php){:target="_blank"}纠缠了很久之后，在[plusls](http://blog.plusls.cn/){:target="_blank"}的思路下做出了这题。然后plusls被suid位的具体作用困惑，同时我对于Linux权限控制的理解也不够，于是花了一点儿时间一起研究了一下，本文做个总结。

<!-- more -->

## Read, Write, eXecute

Unix/Linux文件系统的权限管理是自主访问控制的，一个用户可以授予其它用户某个文件的各种权限，比如读、写、执行，也就是Read, Write和eXecute，分别对应着`ls -l`里出现的`rwx`。

对于目录来说，三者略有不同，`r`变成了能否列出目录中文件的权限，`w`为对目录下文件进行创建、重命名、删除的权限，`x`为进入目录的权限。下面的描述默认以文件为准。

`ls -l`里出现最前面一位表示文件类型，后面紧跟一共三列的`rwx`或者`-`，分别对应着属主的读写执行权限、属组的读写执行权限、其它用户的读写执行权限。而`-`表示不具备该权限。

不考虑最高权限的root，其它用户统统遵循这个权限，即使给自己设置了不可读写不可执行，那也是一样的结果。

而后两列分别是文件或目录的属主和属组。

## 改变权限

Linux下，改变一个文件或目录的权限使用`chmod`命令，基本用法有两种

{% highlight text %}
chmod (u|g|o|a)(=|+|-)(r|w|x|X|s|t) files...
chmod [o]ooo files...
{% endhighlight%}

然而想要改变一个文件或目录的访问权限，当前用户必须是文件或目录的属主，或者是root。

`chmod`第一种形式，是直接说明改变哪些用户组的权限。`u`代表着user，`g`代表group，`o`代表other，`a`代表all。后面跟着`=`，`+`或`-`，表示直接赋予某些权限，增加某些权限或者剥夺某些权限。

而后跟着权限的简写，`rwx`即为读写执行。

`X`是针对目录的`x`，如果对于普通文件添加`X`位是没有用的。尽管对目录`+x`也可以增加访问权限，但是这个方式在针对一个目录下所有子目录添加访问权限，而不想为一些危险的文件添加执行权限时非常有用。

还有两个比较奇特的位将在后面介绍。

多个用户组`chmod`时用逗号隔开，最后面接上需要修改的文件或目录。一个例子是

{% highlight sh %}
chmod u+rwx,g=rw,o-rwx file1 file2
{% endhighlight %}

第二种模式，四个o表示四位八进制数字，第一位可无视，将`rwx`看作八进制数字的三个位，存在`r`则+4，存在`w`则+2，存在`x`则+1。至于其它几种也在下面介绍。

和上面一样的权限，可以这么用

{% highlight sh %}
chmod 760 file1 file2
{% endhighlight %}

当然，也可以改变属主和属组，分别用`chown`和`chgrp`命令即可。这里不再介绍了。

## 粘滞位`T`

粘滞位在各个系统里含义有很大不同，这里只考虑Linux。

Linux中，粘滞位只对目录有效，可以为普通文件添加粘滞位，但是会被内核无视。

其实是一个很好理解的东西，对于目录来说，粘滞位可以保证子目录只能被属主或者root删除。它和`w`的区别就是粘滞位限制了对于目录的删除，而`w`限制了对于文件的。

需要给某个目录添加粘滞位，第一种方法是直接`chmod +t`，或者用`chmod 7777`的方法，第一个数字二进制表示法的第一位表示u，第二位表示g，第三位表示o。

有时候我们能在`ls -l`中看见大写的`T`，这个表示了`x`的状态。因为粘滞位占据了访问位本来的位置，而目录存在访问权限时，为了显示访问权限，粘滞位为小写，否则无访问权限时，粘滞位为大写。

## SUID和SGID

Linux系统中，我们有两种经常打交道的ID，分别是真实ID（RID，Real ID），有效ID（EID，Effective ID）。

SUID和SGID即为文件的`s`位。它的作用为给当前执行该文件的用户赋予文件所有者的有效ID，如果`s`设置在u上即为有效用户ID（这时有效组ID也会变），在g上则为有效组ID。

给文件添加SUID和SGID的方法和目录添加粘滞位是一样的。

SUID和SGID的重要作用，拿经常被用来举例的程序`passwd`来说，Linux中，用户密码被保存在/etc/shadow中，而普通用户对于/etc/shadow没有任何权限，只有root用户有写权限，我们如何做到修改密码，也就是修改了/etc/shadow的呢？

看`passwd`文件，在u的执行位上赫然写着一个`s`。这也就说明我们在执行`passwd`的时候，有着与文件拥有者，即root同等的文件访问权限。这样就可以顺理成章地修改/etc/shadow了。

同样非常常用的一个命令，叫做`sudo`。还有一个命令`ping`其实也是有`S`位的，用于创建ICMP报文。

但是同时，我们的RUID和RGID并不会改变，可以自己测试一下，使用root编译一个调用了`whoami`的子进程，即使设置SUID和SGID，运行后显示出来仍然是执行者，而非文件属主，所以子进程不会继承EID，只会继承RID，这时候可以通过调用几个systemcall来改变自身的ID。

## `setuid`和`setgid`

这是Linux提供的两个systemcall，可以改变程序执行者的RUID和RGID，当然，程序执行者仍然需要权限去修改此权限，一般来说都是SUID SGID和这两个函数配合。实际上`sudo`就是如此的。

同样也有获取当前用户的UID和GID、EUID、EGID以及几个类似的函数，简单列举一下。

{% highlight c %}
#include <unistd.h>

uid_t getuid(void);
gid_t getgid(void);

uid_t geteuid(void);
gid_t getegid(void);

int setuid(uid_t uid);
int setgid(gid_t gid);

int seteuid(uid_t euid);
int setegid(gid_t egid);

int setreuid(uid_t ruid, uid_t euid);
int setregid(gid_t rgid, gid_t egid);
{% endhighlight %}

`get`系列函数，返回的是获取到的ID，`set`系列函数，传入需要的ID，如果返回0则正常，返回-1表示出现异常，并且同时会设置errno。

