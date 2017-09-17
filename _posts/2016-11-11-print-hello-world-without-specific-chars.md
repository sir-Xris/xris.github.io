---
layout: post
title:  C语言不使用特殊字符输出Hello, World!系列
category: C-C++
description: C语言不使用各种字符输出Hello, World!
tags: [ C/C++, 编程 ]
comments: true
---

这套题目来自凝聚工作室招新题，由简至难一共有5题，我这里新增加了学长们提到但是却没有加进去的一题。网络上存在着[6个变态的C语言Hello World程序](http://coolshell.cn/articles/914.html)，大致介绍了C语言的一些**混淆**方法，而此文并不是混淆相关，是对于C语言的冷门知识、旁门左道，甚至可以说是奇技淫巧的运用。

这套题目在我赛后的WriteUp中有提及，觉得还是挺有意义的于是另外开了一篇文章拿了上来。虽然被称之为“奇技淫巧”，不过我想后面的几个都是有助于对于一些基础知识的理解的。

鉴于HW系列题目已经成为凝聚的传统，明年招新的时候大概会撤掉233。

<!-- more -->

**前排提醒，以下代码除非另外提及，均在GCC 6.1版本下可以编译运行。**

## 从零开始的Hello, World!

如果没有任何限制，我想任何人都会输出一句Hello, World!

{% highlight c  %}
#include <stdio.h>
int main() {
    puts("Hello, World!");
    return 0;
}
{% endhighlight %}

## 不使用任何形式的引号 (' ") 输出Hello, World!

因为在C语言中，char也是整型，字符与其编码相等，所以可以直接用编码来表示字符。正常系统都是ASCII码，所以用ASCII码一个一个替换字符即可：

{% highlight c  %}
#include <stdio.h>
char hw[] = {
    0x48, 0x65, 0x6c, 0x6c, // Hell
    0x6f, 0x2c, 0x20, 0x57, // o, W
    0x6f, 0x72, 0x6c, 0x64, // orld
    0x21, 0x00              // !
};
int main() {
    puts(hw);
    return 0;
}
{% endhighlight %}

## 不使用分号 (;) 输出Hello, World!

这个我打在Writeup里的白痴了，重新打了一遍于是习惯性地增加了`using namespace std;`和`return 0;`。

不过无伤大雅，这题的解法是因为`puts`或者`printf`函数有返回值，利用`if`和`while`一类可以使用语句块的语句，作为条件表达式的时候不需要加分号，而最后使用一对大括号来作为`if`或`while`的终结即可。

{% highlight c lineos %}
#include <stdio.h>
int main() {
    if (puts("Hello, World!")) {}
}
{% endhighlight %}

## 不使用井号 (#) 输出Hello, World!

### 0x00 GCC内建函数

GCC有默认包含的`__builtin_`函数，确认过`__builtin_printf`和`__builtin_puts`的存在。

{% highlight c  %}
int main() {
    __builtin_puts("Hello, World");
    return 0;
}
{% endhighlight %}

### 0x01 自行声明函数

这种方法的思路来自于`gcc -E`生成预编译代码，如果偷懒的话直接`gcc -E`然后把生成的代码中去掉一些带有#的行直接拿来用也是可以的。但是这样会多出一堆没用的函数声明，我们只需要其中一个函数，所以只要声明一个即可。

这个经测试在GCC和VS都可用，Clang及其它大概也是可以的，我猜测和C语言的特性有关。

{% highlight c  %}
extern int puts(const char *__s);
int main() {
    puts("Hello, World!");
    return 0;
}
{% endhighlight %}

当然招新题中出的是C++，于是我们要稍作修改。

{% highlight cpp  %}
extern "C" int puts(const char *__s);
int main() {
    puts("Hello, World!");
    return 0;
}
{% endhighlight %}

### 0x02 三字符与双字符组

如同我在Writeup中提到的一般，这个在维基百科的[三字符与双字符组](https://en.wikipedia.org/wiki/Digraphs_and_trigraphs)中有详细介绍。如果是GCC可以直接使用，虽然会有警告，但是在VS中需要设置命令行选项。

下面是三字符组的代码：

{% highlight c  %}
??=include <stdio.h>
int main() {
    puts("Hello, World!");
    return 0;
}
{% endhighlight %}

以及类似的双字符组代码，只有在支持C99以后的编译器才能使用。

{% highlight c  %}
%:include <stdio.h>
int main() {
    puts("Hello, World!");
    return 0;
}
{% endhighlight %}

### 0x03 内联汇编

其实内联汇编是最好想到的，但并不是很好实现的，毕竟还是需要掌握一点汇编的知识。内联汇编的话，也是有两种方法，一个是直接使用`call puts`，另一个是在Linux下使用`syscall`系统调用来实现，本来觉得这里`syscall`还是不用了，但是想到最后一题用了`syscall`的机器码版本，这里还是打上吧。

当然理论上来说最后一题也是可以直接call puts的，不过我懒得再去实验了。

使用`call puts`的代码

{% highlight c  %}
char hw[] = "Hello, World!";
int main() {
    asm("movl   $hw, %edi");
    asm("call   puts");
    return 0;
}
{% endhighlight %}

使用`syscall`的代码

{% highlight cpp  %}
char hw[] = "Hello, World!";
int main() {
    asm("mov    $0x1, %rax"); // syscall调用系统函数编号，这里0x1为sys_write
    asm("mov    $0x1, %rdi"); // sys_write的第一个参数，写入的文件，这里0x1是stdout
    asm("mov    $hw,  %rsi"); // sys_write的第二个参数，字符串地址
    asm("mov    $0xd, %rdx"); // sys_write的第三个参数，字符串长度
    asm("syscall");
    return 0;
}
{% endhighlight %}

## 不使用空格( )及其它相关字符输出Hello, World!

其实就是代码中不能存在`isspace()`返回不为0的字符，举例而言有“\n\r\t\v\f”等等。

这题的思路在SQL注入中经常用到，即使用多行注释来替代空格分隔关键字，当然因为Hello, World!中本身包含空格，我们又需要用到ASCII了，又因为如果包括`#include`的话，不得不回车，又得用到上一题的方法。

不过我在这里为了美观起见（太长了的话……代码里会出现滚动条……），回车了一下，真正写的时候可以删去回车。

{% highlight c  %}
char/**/hw[]={0x48,0x65,0x6c,0x6c,0x6f,0x2c,0x20,0x57,0x6f,0x72,0x6c,0x64,0x21,0x00};
extern/**/int/**/puts(const/**/char*__s);int/**/main(){puts(hw);return(0);}
{% endhighlight %}

## 不使用任何形式的括号\{\}\[\]\(\)\<\>输出Hello, World!

boss题，网上能搜到的资料少之又少，只能在[吾爱破解](http://www.52pojie.cn/thread-280634-1-1.html)和[看雪论坛](http://bbs.pediy.com/showthread.php?t=191367)上搜到一点信息。

好吧，重点是，这两个帖子也是现在凝基的一个学长之前在做凝基招新题的时候问的。~~~哈哈哈，让我笑一会儿~~~

这题还是比较难的，乍看之下，似乎不可做：main函数就至少需要一对括号一对大括号，所以实际上用的已经不是常规方法了。

大致思路就是Shellcode，main是程序的入口，于是我们给main赋值为一段机器码，在运行时就会找到main这个入口点，然后依次执行相应机器码。利用了程序中机器指令和数据值保存在一起的特性（参考：[冯诺依曼架构](https://en.wikipedia.org/wiki/Von_Neumann_architecture)）。

不过因为默认变量是保存在不可执行的.data段，所以我们要不编译选项里增加一句`-zexecstack`，要不就把各变量设为`const`，这时变量会被扔到.rodata段里，是可执行的。

另外值得注意的是，不论是我这里还是OJ上，GCC生成的ELF文件貌似都是小端序，所以我们需要倒着打机器码。

然后以下的代码不仅依赖于编译器，甚至严重依赖于编译器的版本号，我这个是GCC 6.x.x编译出来可以正常执行。

{% highlight c  %}
// 下面两行是Hello, World!的二进制形式
long hw0  = 0x57202c6f6c6c6548; // W ,olleH
long hw1  = 0x00000021646c726f; // ...!dlro
// 下面开始是代码，注释为机器码翻译后的汇编
const long main = 0x9090909090909090; // nop
const long push = 0x9090909090909055; // push %rbp
const long mov0 = 0x9090909090e58948; // mov %rsp, %rbp
const long mov1 = 0x9000000001c0c748; // mov $0x1, %rax
const long mov2 = 0x9000000001c7c748; // mov $0x1, %rdi
const long mov3 = 0x9000600850c6c748; // mov $0x600850, %rsi
const long mov4 = 0x900000000dc2c748; // mov $0xd, %rdx
const long sysc = 0x909090909090050f; // syscall
const long mov5 = 0x90909000000000b8; // mov $0x0, %rax
const long pop  = 0x909090909090905d; // pop %rbp
const long retq = 0x90909090909090c3; // retq
{% endhighlight %}

测试过gcc 5.4.0的hw位置是在0x6010300。不过这种方法应该是有动态寻址，直接找到hw的位置的方法的。

另外就是所有的0x90对应的汇编是nop，所以可以删掉，代码可以压得比较短。

<span style="opacity: 0;">转载请注明出处<a href="http://blog.xris.co/">http://blog.xris.co/</a></span>
