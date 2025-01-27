---
layout: post
title: "app反编译及敏感信息存储"
description: "移动客户端的开发越来越流行，各种app不断出现，但app开发的一些准则慢慢被遗忘，导致各种问题app，其中信息泄漏便是一个问题。"
category: tutorial
tags:
    - decompile
    - ios
    - sso
---
最近在做ios开发，属于刚入门，所以在github上看了一些app的原代码，发现很多代码写法及app设计都存在一些问题，感触比较深的是SINA微博的sso SDK及应用。
### 设计问题 ###
sso ios版的sdk设计有一些问题，竟然要求使用应用从SINA处获得的key及secert，使用key能理解，需要使用secret则比较无法接受，这意味着开发者必须给所有的装有自己app的设备分发secret … 可能因为两者的实现原理不一样，但andriod版的sdk是不需要secert，这才是个合理的设计。
### 反编译案例 ###
[新浪移动应用SSO授权](https://open.weibo.com/wiki/%E7%A7%BB%E5%8A%A8%E5%BA%94%E7%94%A8SSO%E6%8E%88%E6%9D%83)罗列了几个最先使用SSO的应用，因项目需要，想看下他们怎么存储secret的。没有源代码，只能找个应用来反编译了，就拿唱吧…

#### 下载并解压IPA ####
s1. 没装itunes的在这里下载：[唱吧](https://www.appifan.com/app-509885060.html)

s2. 解压

``` bash
unzip ktv.ipa
cd Payload/ktv.app
```

#### 反编译二进制文件 ####
使用otool查看二进制代码，找出可能的字符串常量

```
otool -v -arch armv7 -s __TEXT __cstring ktv > str.txt
vim str.txt
```

**sina的secret是32位的**，所在vim里面 __/[0-9a-zA-Z]\{32}__ ，可以找到几个候选项，再根据key的pattern，__/[0-9]\{10,12}__可以找到key，然后综合考虑下位置就可以找到了。上图为证：
![账号信息](/images/ktv_crake_vim.png)


### 收获感想 ###
1. 敏感或机密信息绝对不能hard code在app代码中
2. 对外发布的sdk尽量设计合理些，避免导致开发商的信息泄漏

### referer ###
1. [otool](https://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/otool.1.html)
2. [MachORuntime](https://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/MachORuntime/Reference/reference.html)
