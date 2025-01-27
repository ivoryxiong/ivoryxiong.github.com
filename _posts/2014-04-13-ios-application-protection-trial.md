---
layout: post
title: "iOS开发总结之软件防护"
description: "OpenSSL的heartbleed漏洞引起业界极大的反响，重新审视安全问题，同样地，App安全也是开发者关注的重点，阿雕也顺道整理下一些App端软件防护的简单措施"
category: devops
tags: 
    - iOS
    - Hopper Disassembler
    - GDB
    - ptrace
---
这几天[OpenSSL](https://www.openssl.org/)的[Heartbleed](https://heartbleed.com/)问题引起业界极大的反响，阿雕也跟风**简单**聊聊iOS下软件攻防。


开发者对安全的理解和安全从业人员理解不一样，开发者只关心软件上的问题，系统层面基本是不会去关心，所以在阿雕眼里，App上的敏感信息没被暴露就是基本的安全。App一般不会存储用户帐户信息，所以敏感信息较少，能想到的大概有：

    1. 用户实时输入的一些信息，比如账户密码，这个主要是保证传输安全；
    2. 开发者相关信息，比如微博、微信等第三方平台账户信息; 
    3. 敏感Api，对于一些安全要求高的应用，App所调用服务器的API也不希望被暴露。

对应的防护措施其实分为传输保护及存储保护，传输保护常用且成熟的方案就是加密，比如采用Https或自定义加密算法【见[iOS开发总结之Https&SSL Pinning](devops/2014/03/02/ios-https-ssl-pinning/)】。而存储保护其实也是加密（不过是变着法子加密），防止用户通过逆向工程获取到信息，另外是防止用户调试，避免被暴力破解（比如密码锁）。

##### 防止逆向工程 #####
逆向工程具体点是反编译，[百宝箱](/toolbox)里提到阿雕用**Hopper**反编译二进制文件,iOS上二进制文件的格式是[Mach-O](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachORuntime/Reference/reference.html)【另见[objc.io的文章](https://www.objc.io/issue-6/mach-o-executables.html)】。IPA里的二进制文件经过Hopper反编译后，能可视化文件的结构，存储的敏感信息如果以字符串常量写在程序中则会暴露无疑. 
看一个例子：
![微博反编译](/images/hopper_weibo.jpg)
从图中可以看到，微博官方客户端的类名及方法暴露无疑，这为hacker调试二进制程序提供很大的便利。但这个解决办法比较简单，将编译选项DEPLOYMENT_POSTPROCESSING设置为YES，Xcode在编译时便会去除调试信息，再反编译将不能如此简单看到程序使用的库、类名、方法。

再看一个[唱吧](/tutorial/2012/11/16/app-decompile-info-store/)的例子，一旦App被破解后，怎么才能尽量少泄漏信息，一个方法是将关键字符串常量混淆，这样即使反编译出来也难于找到真正的字符串（当然，这个措施比较鸡肋），简单点的方式可以采用[UAObfuscatedString](https://github.com/UrbanApps/UAObfuscatedString),将字符串拆解。

##### 防止被调试 #####
GDB是多数人的调试工具，可以使用ptrace阻止GDB依附App进程，具体是调用函数
```
ptrace(PT_DENY_ATTACH,0,0,0);
```

虽然iphone真机没有暴露头文件<sys/ptrace.h>，但依然可以通过dlopen获取ptrace，具体如下:

```
#import <dlfcn.h>
#import <sys/types.h>

typedef int (*ptrace_ptr_t)(int _request, pid_t _pid, caddr_t _addr, int _data);
#if !defined(PT_DENY_ATTACH)
#define PT_DENY_ATTACH 31
#endif  // !defined(PT_DENY_ATTACH)

void disable_gdb() {
  void* handle = dlopen(0, RTLD_GLOBAL | RTLD_NOW);
  ptrace_ptr_t ptrace_ptr = dlsym(handle, "ptrace");
  ptrace_ptr(PT_DENY_ATTACH, 0, 0, 0);
  dlclose(handle);
}
```