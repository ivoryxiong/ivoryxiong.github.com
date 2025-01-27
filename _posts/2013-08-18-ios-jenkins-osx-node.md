---
layout: post
title: "ios开发小结之Jenkins持续构建"
description: "Jenkins是最流行持续构建系统，广泛应用于各种类型的工程。通过其Master-Slave的结构，稍作配置也可用于ios工程的持续构建"
category: devops
tags:
    - jenkins
    - ios
    - master-slave
    - ssh
    - scp
---

之前在做广告系统和Java Web开发时一直使用Jenkins做持续构建，效果不错，能比较及时发现一些提交代码中的问题。由于Java的跨平台性，没有仔细了解过Jenkins的Master-Slave特性，但ios开发要求OSX系统，Jenkins是运行在Linux Server上，所以需要给运行中的Jenkins添加使用OSX系统的node并配置对应Job，同时为了让产品同学尽早看到效果，也需要把构建出的IPA传到一个共享的位置。

### 添加OSX node ###
Jenkins官网的[Step-By-Step教程](https://wiki.jenkins-ci.org/display/JENKINS/Step+by+step+guide+to+set+up+master+and+slave+machines)比较详细地介绍了添加Node的过程，需要注意的几点是：

1. 最好新建一个jenkins用户，将"Remote FS Root"设置为其home目录/Users/jenkins
2. Usage选"Leave this machine for tied jobs only"
3.  Launch Methods 教程建议选Java Web Start，理由是Master不需要知道Slave Node的地址，不过对于公司内部网络而言，这些配置都比较固定，而且很多会用https协议替代http，所以建议使用通过ssh启动。
4.  配置SSH，使两台机器之间SSH登录无需密码，具体步骤见：[SSH login without password](https://www.linuxproblem.org/art_9.html)。生成key后，在"系统管理"-> "Manage Credentials" 里Add Credentials，并选"SSH Username with private key"

配置好全部后，直接在"系统管理" -> "管理节点"中选中新添加节点并启动

### 配置ios对应job ###
得益于Jenkins的插件机制，配置build Xcode工程的Job可以使用[Xcode Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Xcode+Plugin),只需很少的几步。不过由于自己本省要build的工程中便带有一个Shell脚本build.sh，所以没有采用Xcode Plugin配置。不过新建Job的流程都一样，只有很小差别，关键几步如下:

1. "新Job"里创建一个类型为"构建一个自由风格的软件项目"的任务
2. 配置里勾选"Restrict where this project can be run", 并将"Label Expression" 配置为OSX节点的label
3. "构建"里选Execute shell，并输入对应构建命令，如
``` 
${WORKSPACE}/build.sh dis 9.9.9
```
对应Xcode Plugin，构建里选"Xcode",并做对应配置(唯一比较麻烦的就是"Keychain path"，需要导入对应的认证文件以支持IPA签名)。

配置好后，直接构建一次，看看效果如何吧

### 发布构建好的IPA ###
这个比较多方法，可以放到ftp，也可能放到NFS上，又或者特定的某台机器。我选择的是将通过SCP将构建的IPA拷贝到内部共享的一个服务器上。这也分几步：

1. 从"系统管理" -> "管理插件" 安装[SCP插件](https://wiki.jenkins-ci.org/display/JENKINS/SCP+plugin)
2. 在"系统管理" -> "系统设置"里的"SCP repository hosts"模块里配置SCP插件，新建一个SCP site "tmp"
3. 配置ios对应Job，增加一个"构建后操作",选择"Publish artifacts to SCP Repository", 并设置SCP site为tmp，然后设置对应文件源及目的地，如Source为"ipa/*.ipa"， Destination为"iPhoneApp/"。


到此为止， 可构建ios工程的Jenkins便配置好了，赶紧push个提交看看效果去！

#### Reference: ####
1.  [Jenkins Wiki & Plugins](https://wiki.jenkins-ci.org/)
2.  [Linux Problems](https://www.linuxproblem.org/) 