---
layout: post
title: "ios开发小结之工具优化"
description: "古话说的好，工欲善其事必先利其器，做ios开发也一样，本文总结下ios开发中的工具优化"
category: devops
tags:
    - software engineering
    - ios
    - ide
    - develop tools
    - git
    - unit test
    - ocunit
    - hgunit
---
ios的开发工具集比较单一，Apple都将它打包在Xcode中，所以想提高效率，必须熟悉Xcode，必要时还需对其做些优化。

先说说编辑器，主流的代码编辑器Vim，XCode，Sublime，TextMate。XCode自带编辑器还不错，但在format、替换查找方面做得不够好，比如没有像eclipse一样重排import语句（间接说明了开源力量的强大）。不过这也不是没办法解决，用shell和automator就能比较完满地解决，详见[stackoverflow](http://stackoverflow.com/questions/1905347/a-way-to-automatically-organize-imports-in-xcode).在这个解决方案中，format代码时shell完成的，而automator只是做集成，如果用格式化程序uncrustify替换shell，便能大大增强XCode的功能。关于如何将uncrustify集成到XCode中详见：[github
](https://github.com/bengardner/uncrustify/),配置过程中注意下uncrustify的路径问题，为了保证可用，可以考虑将编译好的uncrustify可执行文件copy到/bin下，另外编写好services后在automator里运行测试一次。 对于VIM流而言，在XCode中编辑的效率降低不少，但不断在VIM和XCode中切换也很麻烦（当然也可以用XCode CLI），不过万能的VI控们开发了XCode插件[XVim](https://github.com/JugglerShu/XVim)。

SCM是开发必备工具之一，XCode继承了先进好用的GIT（当然HG也很不错，SCM的对比见[Why Git is Better than X](http://thkoch2001.github.com/whygitisbetter/)）。git的CLI做得很不错，比如命令 “git log  --oneline --graph” 能将当前分支上的代码修改历史用ASCII字符画出图形（十分geek，十分便捷）。当然对于我等小白而言，开始命令行参数掌握的还是不够多，使用GUI也是一种不错的选择，[Source Tree](http://www.sourcetreeapp.com/)便是一个不错的选择。

代码静态分析是保证代码质量中重要的一步，XCode自带了[Clang](http://clang-analyzer.llvm.org/)静态分析工具，在XCode中product=>analyze便可对工程进行分析。除Clang外，[OCLint](http://oclint.org/)也是一个不错的工具，提供了一些Clang不具有的功能，比如重复代码、坏代码味道，很值得一试。Unit Test是代码逻辑正确的重要保证，XCode在创建工程时选择include unit test则会自动声称一个test target，运行时也只需要选择对应的Scheme就ok。XCode自带的单元测试框架是OCUnit，[GHUnit](https://github.com/gabriel/gh-unit)则是另一个常用的测试框架，也值得试试。









