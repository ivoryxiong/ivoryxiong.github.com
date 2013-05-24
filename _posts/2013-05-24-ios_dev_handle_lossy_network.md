---
layout: post
title: "ios开发小结之低速网络处理"
description: "在目前国内运营商提供的网络质量下，低速网络处理是一个优质app必须面对的问题，模拟低速网络和异常处理是其中关键。"
category: devops
tags:
    - 低速网络
    - ipfw
    - 异步请求

---
{% include JB/setup %}

app开发在第一个版本完成后，最主要的工作便是迭代，最近两个sprint(我们使用scrum做开发迭代)的主要任务都在完善一些细节及bug fix。

app在低速网络的表现在做功能开发的时候考虑不够，表现很糟糕。解决这个问题的首要步骤便是模拟低速网络，如果喜欢图形界面，[Network Link Conditioner](http://mattgemmell.com/2011/07/25/network-link-conditioner-in-lion/)  是一个不错的选择。对于喜欢自由和脚本化的码农而言，GUI提供的功能很有限而且用起来效率低下（比如GUI的设置是针对机器的，也就是说本来只想模拟器的网络差点，结果一设置，邮件收不到了，google上不去了...），庆幸的事OSX是基于Unix的BSD系列，自带有[ipfw](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/ipfw.8.html )神器。freebsd上关于[ipfw](http://www.freebsd.org/doc/zh_CN/books/handbook/firewalls-ipfw.html)有详细的文档介绍，模拟低速网络只用到其中很简单的TRAFFIC SHAPER功能。app的网络请求是和指定的服务器进行通信，所以使用ipfw对某一服务器的网速进行控制便可: 

{% highlight bash linenos %}
sudo ipfw add pipe 1 ip from any to www.ivoryxiong.org out
sudo ipfw add pipe 2 ip from www.ivoryxiong.org to any in
sudo ipfw pipe 1 config delay 250ms bw 16Kbit/s queue 4Kbytes
sudo ipfw pipe 2 config delay 250ms bw 16Kbit/s queue 4Kbytes
{% endhighlight %}

简单解释下： 通信链路是双向的，所以1-2行分别创建了出和入流量的两个管道,3-4行分别设置了两个管道的延时、带宽及队列大小。执行这段脚本的效果如下图:
![ipfw](/imgs/ipfw.png)

设置低速网络后，从截图可以看出模拟出的效果还不错，那么剩下的问题就是如何用代码处理因低速网络产生的问题。在网络慢的情况下，用户可能进入一个页面后等很久也刷新不出来便回退了，如果请求是异步的，那么可能request没有被及时取消，而请求完成时调用的delegate其实已经被dealloc，crash!! [Objective-C: delegate的那点事儿](http://longtimenoc.com/archives/objective-c-delegate%E7%9A%84%E9%82%A3%E7%82%B9%E4%BA%8B%E5%84%BF)详细地描述了这个问题，[stackoverflow](http://stackoverflow.com/questions/2720087/handle-when-callback-to-a-dealloced-delegate)上也没有好的解决办法，看来只能从程序结构上解决这个问题。

一个可行的解决办法是异步请求在创建的时候给一个cancel的notifiaction name，在收到对应notificaton后便不再回调。而所有notification的name统一管理，比如写在viewcontroller的基类里（请求多数是在viewcontroller里发出的），每个viewcontroller instance的notification应该是一样的，在viewWillDisappear时post notification。如果request在view里发出的（当然从MVC模式讲，这已经不合理了），那么对应的notification name也应该和包含它的viewcontroller的一致，如果不一致，对view而言，post notification的时间点不好把握。

