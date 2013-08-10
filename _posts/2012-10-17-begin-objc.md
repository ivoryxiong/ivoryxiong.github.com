---
layout: post
title: "begin objc with gnustep"
description: ""
category: devops
tags: [objc]
---
## install gnustep ##

在linuxMint里，直接执行命令:

``` bash
sudo apt-get install gobjc gnusetup gnusetup-devel
```

然后在.bashrc里加入下面代码并source

``` bash
/usr/share/GNUstep/Makefiles/GNUstep.sh
```

## hello world ##
创建文件 hello.m

``` objc
#import <Foundation/Foundation.h>

int main (int argc, const char * argv[]) {
    NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];
    NSLog(@"hello world!");
    [pool drain];
    return 0;
}
```

建立编译文件GNUMakefile

``` bash
include $(GNUSTEP_MAKEFILES)/common.make
TOOL_NAME=hello
hello_OBJC_FILES=hello.m
include $(GNUSTEP_MAKEFILES)/tool.make
```

文件中的GNUSTEP_MAKEFILES变量在安装gnustep时已经设置，hello是编译目标，对应的代码文件是hello.m，其中hello_OBJC_FILES的hello必须和TOOL_NAME一致。

make 后会发现多了一个obj文件夹，里面的hello便是可执行二进制文件，执行结果如下：

``` bash
ivoryxiong@ivoryxiong ~/github/objc-learn/basic $ ./obj/hello
2012-10-18 00:24:23.947 hello[7241] hello world!
```
