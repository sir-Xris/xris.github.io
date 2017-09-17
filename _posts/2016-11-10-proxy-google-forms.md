---
layout: post
title: 代理Google Forms
modified: 2016-11-10
category: backend
description: 'use nginx to forward proxy google forms'
tags: [ Nginx, 运维 ]
comments: true
---

因为最近要写英语作业，设计一份问卷调查，我们小组里我自己负责了网络问卷调查的部分。当我在Google Forms上折腾问卷调查了折腾半死终于搞好，准备着publish到自己博客上的时候，突然发现Google Form提供的有限三种发送方式只包括邮件发送、固定链接和`iframe`实现的嵌入HTML。似乎并没有什么好的方法可以让我墙内的同学们填上表。于是考虑到Nginx本身就基于代理，就打算另外搞个页面来做代理，从VPS转发Google Forms到这边。

<!-- more -->

首先嘛，要做一个这样的页面，Nginx的模块ngx_http_sub_module或者更强大的ngx_http_substitutions_filter_module是必须的，我们用这个工具来修改请求、响应的内容，把所有向Google的请求修改成向代理网址的请求。如果是`apt install`安装的Nginx，现在只需要另外`apt install nginx-extra`就装上了一些包括上述的常用模块，如果是手动编译的，那么还需要重新编译一遍。

有了上面的工具之后，想搞一个代理页面还是很简单的，可以选择新建一个子域名，或者用一个“子目录”。我是去DNS那边添加了个hw.xris.co子域之后在Nginx的配置文件中增加一个server，当然之前试过子目录也是可以的，可以选择别的`server`新增一条`location`记录来`proxy_pass`。

不过因为以前没有怎么折腾过Nginx，在配置上还是花了不少功夫，踩了不少坑，所以也稍微记录一下。首先是最简单的，直接转发页面：

{% highlight nginx %}
server {
    listen 80;
    listen [::]:80;

    server_name ${your_site};

    location / {
        # 此处${site_of_form}是Google给你的网址，要去掉viewform
        # 要注意的是：不能用短网址，因为短网址会被直接重定向
        proxy_pass      ${site_of_form};
        proxy_redirect  off;

        subs_filter     ${site_of_form} ${your_site};
    }
}
{% endhighlight %}

上面的`${site_of_form}`，从Google Forms给的url中去掉最后的viewform就是我们需要的。因为Nginx代理的工作原理十分粗暴简单，直接把`${site_of_form}`作为`${your_site}`的root，所以如果在`${site_of_form}`中点了一个跳转到另一个子目录的超链接，会无法触发subs_filter的过滤效果导致跳转，并且也无法直接访问该目录。

巧的是，实际上Google Forms的页面是会在点了第一个下一步之后跳转的，所以无话可说了，必须如此。

如此改了之后，发给别人的页面url应该是${your_site}/viewform。

测试，打开这个页面发现~~卧槽这是什么玩意儿~~，显然样式表没有加载好，检查浏览器控制台可以发现这么两段

{% highlight http %}
GET http://fonts.googleapis.com/blablabla net::ERR_CONNECTION_RESET

XMLHttpRequest cannot load https://www.gstatic.com/blablabla.
No 'Access-Control-Allow-Origin' header is present on the requested resource.
Origin 'http://hw.xris.co' is therefore not allowed access.
{% endhighlight %}

看起来我们需要继续去代理fonts.googleapis.com和www.gstatic.com。

font.googleapis.com国内代理还是非常好找的。而gstatic我粗略看了一下似乎没有，于是我打算自己再去搞个代理，又跑到DNS中添加了一条记录，接着修改Nginx配置文件，增加了一条`server`。

{% highlight nginx %}
server {
    listen 80;
    listen [::]:80;

    server_name gstatic.xris.co;

    location / {
        proxy_pass      https://www.gstatic.com;
        add_header      Access-Control-Allow-Origin *;
        subs_filter     //www.gstatic.com //${proxy_for_gstatic};
    }
}
{% endhighlight %}

如果愿意做雷锋向世界提供代理的话，这样就可以了。如果服务器本身流量不多，不希望别人使用自己的代理，可以在`location / {}`里增加一条：

{% highlight nginx %}
valid_referers server_name ${your_site} *.${your_site};
if ($invalid_referer) { return 404; }
{% endhighlight %}

然后对于`${your_site}`的`server`再做更新，添加gstatic和google fonts的subs_filter，我们就完成了第一个页面的代理。

**才怪，你会发现一点用都没有。**

原因是subs_filter无法处理压缩过的网页，所以还要在原页面的`server`中添加HTTP头，告诉Google我不要压缩过的网页，也就是`proxy_set_header Accept-Encoding ""`。

于是就变成了下面这样：

{% highlight nginx %}
server {
    listen 80;
    listen [::]:80;

    server_name ${your_site};

    location / {
        proxy_pass          ${site_of_form};
        proxy_redirect      off;
        proxy_set_header    Accept-Encoding "";

        subs_filter         ${site_of_form}         ${your_site};
        subs_filter         //www.gstatic.com       //${proxy_for_gstatic};
        subs_filter         //fonts.googleapis.com  //${proxy_for_gfonts};
        subs_filter_types   text/css text/javascript;
    }
}
{% endhighlight %}

最后提示一下subs_filter的原理也是简单粗暴，把所有接收到的信息里的匹配的字符串全部替换成目标字符串，所以你可以想到所有正文里的字符串也将会被替换。所以一般情况下还是建议少使用subs_filter。

另外提一句题外话，我的代理页面的hw.xris.co里的hw是HomeWork，不是Hello World。

UPDATE: 作业已完成
