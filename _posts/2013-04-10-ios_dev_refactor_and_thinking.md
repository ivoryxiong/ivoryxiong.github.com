---
layout: post
title: "ios开发小结之重构及随想"
description: "对于团队开发，代码需要保持长期重构，否则长期的技术债务会压垮整个团队！"
category: devops
tags:
    - ios
    - xib
    - refactor
---
### 抛弃xib ###
最近项目工期比较宽裕，做了比较多refactor的事情，最重要的一件事情是把项目中的很大一部分的nib/xib文件移除了。关于项目中是否应该使用IB的争论一直很多（如[reddit](http://www.reddit.com/r/programming/comments/15jjfi/why_i_dont_use_interface_builder/)），也没有好的结论。但对于团队开发而言，拒绝IB无疑是一个明智的选择，至少从目前的经历而言，使用nib/xib有以下不足之处：
1. 不方便团队其他成员进行review，nib/xib文件是采用xml，目前主流的review工具（如[reviewborad](http://reviewboard.org)）支持不够好，比较难看明白。
2. nib/xib的xml Scheme定义比较复杂，而且带有ide版本等信息，一个小的修改往往导致一大堆的无关diff，review、merge等开发流程基本无效。
3. 比较难复用，在[无觅阅读](https://itunes.apple.com/cn/app/id593518746?mt=8)中，动态有多种类型，每种类型样式不同，但都有同一的用户信息，如果用xib/nib的话，可能就只能每个类型都单独写一遍，导致大量重复代码，重复代码对于工程来说是致命的质量问题（见[重构-改善既有代码的设计](http://book.douban.com/subject/4262627/)）,在这部分代码有修改时极易引入bug。
   
外包引入的xib/nib文件问题对我们项目影响很大，在改版的时候几乎都要把人整崩溃（每个xib都机械地修改一遍back按钮）。建议如果是团队开发而非小作坊，而且关注代码质量，有支持迭代打算，抛弃xib/nib吧！
   
### app升级测试 ###
app升级测试是最近感触比较深的一个问题，特别是应用大改版时。在几次升级的过程中总结出了几点：

1. 测试要针对所有的已发布版本（难度有点大，特别是早期版本，尽量吧），否则如果有纰漏被用户卸载可不好玩。

2. 如果版本涉及到本地存储方式的修改，尽量兼容之前的版本，实在搞不定可以重置这些数据，但一定要提示用户（虽然大多数数据时缓存数据和登录信息）。 

### 其他小问题 ###
1. 需要注意ios对文件名长度有限制，因为这个曾导致应用的图片缓存失败！通用的解决方式对原文件名称hash后存储，[SDCWebImageView](https://github.com/rs/SDWebImage/)里面有一个很好的算法,可以参考下。
2. 系统版本兼容问题,ios5以下AutoReleasePool只在主线程里有，其他线程不会自动派生，而ios5以上的会自动派生。现在都在ios5以上开发，但如果安装到了ios4.x的设备上可能会导致内存泄漏。
3. 图片资源需要统一管理，单例的Manager进行管理，方便支持换肤等功能。
   

