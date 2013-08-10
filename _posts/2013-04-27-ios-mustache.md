---
layout: post
title: "ios开发小结之模板引擎mustache"
description: "计算机一个最大的贡献便是替人类完成重复的脑力活动，对于语言而言，循环是最直接的见证，而对于程序和工程而言，类似的便有模板引擎！"
category: devops
tags:
    - mustache
    - template engine
---
模板引擎（template engine） 对于开发者而言都不陌生，常用的有FreeMaker、Velocity、Smarty等（详见[wikipedia](http://en.wikipedia.org/wiki/Template_engine_\(web\))），多用于动态渲染web页面。用于渲染web页面的模板引擎功能方面都比较强大，支持大部分的逻辑，可以说是一门小型语言。在web渲染的应用场景外，还有许多场景需要用到模板引擎，比如运维人员向服务器集群分发具有目标机器信息的配置、广告系统根据用户配置实例化不用的广告样式（好吧，其实这个是web应用…）等，在这些场景下，可能需要的只是替换设定的变量再加上点简单的判断逻辑，那么FreeMaker、Velocity显得就大材小用。

[mustache](http://mustache.github.io/)便是一个适用于上述场景的简单、轻量的跨平台的模板引擎，只提供简单的替换、循环、判断等必要功能。由于逻辑简单，基本上5分钟之内便可以拿来用，而且主页有一个[demo](http://mustache.github.io/#demo),马上拿来用吧！ 包括变量在内，mustache只支持[6种tag](http://mustache.github.io/mustache.5.html)（够简单吧:)），虽然简单，但基本覆盖了常用的功能，而且支持Lambdas可以算是一个很大的亮点吧！

   最近在ios项目中是使用mustache根据用户配置生成对应的xcode project中的项目文件app-info.plist和自定义的配置文件config.plist。这些工作是在build project之前做的，所以写了一个python脚本使用mustache生成对应文件，附上作为参考：

``` python
import pystache
import json
import sys

def renderf(f, config):
    mtemplate = open(f+".mustache", 'r').read()
    plist = open(f, 'w')
    plist.write(pystache.render(mtemplate, context).encode('utf-8'))
    plist.close()

if __name__ == '__main__' :
    if len(sys.argv) < 3:
        print 'input template file'
        exit

    json_data=open(sys.argv[1])
    context = json.load(json_data)
    json_data.close()

    for arg in sys.argv[2:]:
        renderf(arg,context);
```

其中[pystache](https://github.com/defunkt/pystache)是官方推荐的python包，使用也方便。

   当然，如果实在是中意shell，也可以bash脚本完成这些替换，见[bash-template-engine](https://github.com/napsternxg/bash-template-engine),但只能完成最基本的替换，循环什么的就无能为力了。
