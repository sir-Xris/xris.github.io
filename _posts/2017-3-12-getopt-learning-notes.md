---
title: getopt学习笔记
layout: post
category: learning-notes
tag: [ getopt, 学习笔记 ]
comments: true
---

绝大部分时候，在命令行里调用一个程序，我们都需要给这个程序传入参数，也就是一般而言出现在（比如说）C的main函数参数里面的`int argc, char *argv[]`这玩意儿。举个例子，在使用`ls`的时候，我们可能会这么用`ls -l --color=auto`，为了确认传入的argv，我们需要另外一段代码来处理。

以人类懒惰的天性——这代码明显能复用，为什么不写个可复用的代码呢？于是便有了getopt这个东西，可以帮助分析传入程序的参数。还是拿上面那条`ls -l --color=auto`举例，这里返回的是一个`'l'`和一个其实是ls.c里定义的enum值：`130`，这个具体还是见ls.c源码比较好。

getopt有三个版本：`getopt`、`getopt_long`和`getopt_long_only`，上面`ls`的一个长参数`color`就是用`getopt_long`实现的，下面会讲使用方法。至于具体代码，可以去看getopt的源代码。

[GNU家的getopt](https://www.gnu.org/software/gengetopt/gengetopt.html)

[BSD家的getopt](https://github.com/freebsd/freebsd/tree/master/lib/libc/stdlib)

不过本人长期沉迷GNU，所以没有研究过BSD家的。这里是[GNU家的文档](https://www.gnu.org/software/libc/manual/html_node/Getopt.html)。

<!-- more -->

## 起因

大概在一个多月前，调戏shell下的工具`yes`的时候突然想到：如果我想输出一个-x，那我应该如何输入参数？这算是一切的起因了。

一开始以为只是个小case，试了一下`yes -x`，报错：无效的选项。然后试了试`yes \-x`，同样报无效的选项错误。继续用`yes "-x"`、`yes "\-x"`以及它们的单引号版本，前者报错后者输出一堆`\-x`。

事情并没有我想象中的那么容易呢……于是我又换了个姿势，`echo -x | xargs yes`，报错，`yes $(echo -x)`，报错。

完了，我没招了，不如进`man`看看吧？ 结果，手册里没有给出任何的输出带一个减号的方法，只有两个正经的参数`--help`和`--version`，我摔！

于是去Google，未果，Stackoverflow，未果，最后去问了各位前辈……

一开始理我的是whtsky，他在他的Mac上试了试`yes -x`，果断输出了一长串的`-x`，于是看了看版本，发现是BSD套件，我看了一眼我的，果然是GNU套件。于是就被安利了Mac。我穷买不起啊摔！

然后wind表示快换Mac吧，再摔！

后来w学长（三位w姓学长）告诉我用`yes -- -x`可以，我试了试果然成功，但是为什么呢？w稍微解释了一下getopt里`--`是终止处理的含义，并扔给我了一段getopt的源码，于是顺便就将整个`getopt`研究了一遍。

## getopt

*为了区分运行程序时传入的参数和参数的参数，我下面将运行时传入的带-号的参数称为"选项"，选项的参数就称为"选项参数"，其余非选项、非属于某个选项的参数的传入参数就叫参数*。（就是这么简单粗暴）

只要#include了&lt;getopt.h&gt;，就有几个全局变量可以用。

1. `int opterr`，由用户设置，默认为0。如果该值非0，`getopt`函数在遇到未声选项数的时候会向标准输出`stderr`打印一些错误信息；如果为0则不会打印。

2. `int optopt`，由函数设置。如果遇到未声明选项，该值会被设置为这个未声明选项。

3. `int optind`，由函数设置。值为下一个需要处理的传入选项的下标。

4. `char *optarg`，由函数设置，在一些需要参数的选项里，返回为选项参数所在字符串的指针。

函数`getopt`的原型是

{% highlight c %}
int getopt(int argc, char *const *argv, const char *shortopts);
{% endhighlight %}

第一个参数`argc`为选项个数，第二个参数`argv`为各个选项的指针，第三个参数`shortopts`声明了所有应被`getopt`接受的选项的形式。

前两个还好说，第三个是需要一定格式的。每个选项的后面可以接`:`，表示选项必须要有一个选项参数，或者接`::`，表示这个选项后的选项参数可有可无，如果不接任何冒号，则表示不接受任何选项参数。

每次处理完一个声明的选项，`getopt`都会返回这个选项的值，如果有选项参数，则设置`optarg`。如果没有某个减号带头的程序参数没有被声明，则返回`'?'`。所以我一开始`yes -x`的`-x`就被当作一个非法选项了。

`getopt`有三种处理传入参数的方式：

1. 默认情况下，`getopt`会优先处理选项，所有非选项的传入参数都会被扔到最后不会处理。当然，在这种情况下你传入的`argv`会被重排。

2. 如果`shortopts`开头是一个减号（-），依次处理传入参数，所有的非选项传入参数都会被`getopt`返回一个`'\x01'`，并设置到`optarg`里。

3. 还有一个模式是由POSIX标准规定的，如果`shortopts`开头是一个加号（+）或者设置了系统变量`POSIXLY_CORRECT`，那么依次处理传入参数，遇到一个非选项的参数就停止处理接下来的参数。BSD套件也默认是这种模式，于是这也就解释了为什么whtsky的可以正常执行。

举例更好理解。

{% highlight c %}
#include <ctype.h>
#include <getopt.h>
#include <stdio.h>

opterr = 0;

int main(int argc, char *argv[]) {
    int option;

    while ((option = getopt(argc, argv, "ab::c:")) != -1)
        switch (option) {
        case 'a': puts("option a"); break;
        case 'b': printf("option b\targv %s\n", optarg); break;
        case 'c': printf("option c\targv %s\n", optarg); break;
        case '?':
            if (optopt == 'c')
                fprintf(stderr, "option %c requires an argument.\n", optopt);
            else if (isprint(optopt))
                fprintf(stderr, "unknown option `%c'.\n", optopt);
            else fprintf(stderr, "unknown option char `\\x%x'.\n", optopt);
        }

    for (; optind != argc; ++optind)
        printf("non-option argument %s\n", argv[optind]);
    return 0;
}
{% endhighlight%}

为了不被当作一个非选项的程序参数，可选参数选项的参数和选项之间不能有分隔符，如空格。

当用`./program -ab1 -c 2 -b 3 -d 4`运行程序，输出为

<pre>
option a
option b    argv 1
option c    argv 2
option b    argv (null)
unknown option `d'.
non-option argument 3
non-option argument 4
</pre>

当然，可以自己改一下`getopt`的参数玩玩。

## getopt\_long

`getopt_long`就比`getopt`要复杂多了。除了兼容`getopt`的部分，为了声明每个长选项，还另外有个结构体`struct option`，含有下列成员：

`const char *name`，长参数名，一个字符串。

`int has_arg`，是否有选项参数，三个可能的值：`no_argument`，`required_argument`和`optional_argument`。

`int *flag, val`，这个略复杂，决定了`getopt_long`捕获一个选项时该如何处理。

当`flag`是一个空指针时，val是`getopt_long`在捕获长选项时返回的值。上文的`ls`其实就是这么处理的，它将`val`设置为`130`了罢了。这个值可以多个长选项重复或和短选项重复。

当`flag`非空，在`getopt_long`捕获一个长选项时会将`&flag`赋值为`val`。

函数原型为

{% highlight c %}
int getopt_long(int argc, char *const *argv, const char *shortopts, const struct option *longopts, int *indexptr);
{% endhighlight %}

前三个和`getopt`没有区别，`longopts`是上面说的`struct option`，`indexptr`则变成了一个指针，`indexptr`这个变量初始值为`getopt_long`首先要处理的`argv`下标，开始第一次之后，如果遇到`has_arg`不为`NULL`的一个长选项，会被赋值为它在`longopts`里的下标。

同样的，可选参数的选项如果有参数时，短参数中间不能有分隔符，长选项和参数之间要仅有一个等号（=）分隔。

可能没有讲完，不过举个例可以清楚很多。还有不理解的可以自己改程序玩玩。

{% highlight c %}
#include <getopt.h>
#include <stdio.h>

int length = 0;

struct option longopts[] = {
    {"short",       no_argument,        &length,     0 },
    {"long",        no_argument,        &length,     1 },
    {"all",         no_argument,        NULL,       'a'},
    {"auto",        no_argument,        NULL,       'a'},
    {"color",       optional_argument,  NULL,       128},
    {"file",        required_argument,  NULL,       'f'},
    {NULL,          no_argument,        NULL,        0 }  // keep the last one 0
};

int main(int argc, char *argv[]) {
    int last_index = 1, index = 1, option;

    while (~(option = getopt_long(argc, argv, "abf:", longopts, &index)))
        switch (option) {
        case 'a':
            if (index != last_index) {
                last_index = index;
                printf("option %s\n", longopts[index].name);
            } else puts("option a");
            break;
        case 'b': puts("option b"); break;
        case 'f':
            if (index != last_index) {
                last_index = index;
                printf("option file\targv %s\n", optarg);
            } else printf("option f\targv %s\n", optarg);
            break;
        case 128: printf("option color\targv %s\n", optarg); break;
        case '?': /* getopt_long printed an error message */ break;
        }

    printf("option %s\n", length ? "long" : "short");

    for (; optind != argc; ++optind)
        printf("non-option arguments %s\n", argv[optind]); 
    return 0;
}
{% endhighlight %}

如果用如下命令执行程序`./program --long -all --color=auto --file filename inputfile`，输出为

<pre>
option all
option color  argv auto
option file   argv filename
option long
non-option arguments inputfile
</pre>

## getopt\_long\_only

经常可以见到一个减号后跟着长选项的，举例说gcc的`-std=c++17`，这就是由`getopt_long_only`做到的。

`getopt_long_only`原型和`getopt_long`一模一样，唯一的区别在于`getopt_long_only`在遇到一个减号后接一串字符的时候会先检查`longopts`里有没有，没有再作为短选项处理。

直接举例不多说话：

{% highlight c %}
#include <getopt.h>
#include <stdio.h>

struct option longopts[] = {
    {"std", required_argument, NULL, 'S'},
    {NULL,  no_argument,       NULL,  0 }
};

int main(int argc, char *argv[]) {
    int index = 1, option;
    while (~(option = getopt_long_only(argc, argv, "s:", longopts, &index)))    
        switch (option) {
        case 's': printf("option s\targv %s\n", optarg); break;
        case 'S': printf("option std\targv %s\n", optarg); break;
        case '?': break;
        }
    return 0;
}
{% endhighlight %}

输入为`./program -saber --std=c++14 -std=c++17 -saber`时候，输出为

<pre>
option s   argv aber
option std argv c++14
option std argv c++17
</pre>

