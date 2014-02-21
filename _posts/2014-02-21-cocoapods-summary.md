---
layout: post
title: "iOS开发小结之cocoapods有感"
description: "分享使用cocoapods的一些体会，集中在迁移工程中走的弯路"
category: devops 
tags:
    - cocoapods
    - Podfile
    - ios
---
与众多iOS开发者一样，刚开始通过copy代码的方式引入第三方开源库到工程中，而对于自己维护的私有库，则通过submodule与subproject的方式维护。这样做的弊端很明显: 首先，不能及时更新第三方开源库的代码；其次，在主工程里容易形成多层工程嵌套工程的结构。作为一名有Java背景的开发者，在类Maven的cocoapods出现后，真是欣喜若狂，本文和朋友们分享一些使用经历及经验。

### 环境搭建
具体搭建过程就不多讲了，正常参照官网的指引不会有大问题。 唯一需要理解的是CocoaPods的工作原理，既能方便解决使用中的问题，又能搭建私有Specs Repo。其实CocoaPods的原理很简单，和Maven也类似，总体上分两部分：如何维护第三方库及如何集成第三方库。

* ##### Specs Repo 维护类库信息
cocoapods 官方使用[Specs](https://github.com/CocoaPods/Specs)来维护类库信息， specs repo按照一般的目录结构保存类库的元信息(Specs)。 安装cocoapos后用户执行pod update命令，cocoapods会在用户主目录下创建文件夹 .cocoapods, 并从github上将[Specs](https://github.com/CocoaPods/Specs) clone到目录.cocoapods/repos/master。pod中大部分命令【如list, search】，如果没有指定数据源，都会以.cocoapods/repos/作为数据源, 所以要是有私有类库在团队内共享，可以建一个私有的specs repo， 具体步骤参见[官网说明](http://guides.cocoapods.org/making/private-cocoapods.html)。  

* #### 新建xcode poject集成类库
编辑好Podfile(与Example.xcodeproject在同一目录)后使用命令pod install给工程集成第三方类库。cocoapods将project转换成workspace， 并在该workspace下新建project Pods, 所有依赖的代码都会被clone下来后copy到Pods工程。具体到Pods工程，cocoapods为每个依赖设置了一个target, 同时也设置一个合并了所有依赖的target， 原有工程也就依赖这个target生成的对应lib。workspace及Pods工程更多的细节不是本文的重点，参考阅读资料。 

### 工程迁移

将工程迁移到cocoapods是使用过程重要的一步，博主对cocoapods的知识基本都来自官网，所以一开始走了弯路:(，但[making a cocoapod](http://guides.cocoapods.org/making/making-a-cocoapod.html)介绍的方法并不十分适合用来做迁移。 

* #### 模板结构
如果我们新建一个如官网模板的pod，维护pod时会发现一个问题是，由于Classes下面的源文件不属于任何 Xcode project，将很难对它们进行修改【编译，测试都难以完成】。也许有人会说，Project【以前版本时Example】目录里的工程在pod install后可以修改呀，但可别忘记，Project下Pods工程的源文件时copy过去的，也就是无法做diff，而且这样做还无法更新其他依赖的库【pod update后developement pod会被覆盖】，而且即使最后修改完了，还得copy到Classes目录。

* #### 实用结构
博主最后采用的方式是原有工程结构保持不变，直接在目录下添加ProjectName.podspec与Podfile。添加podspec文件是因为最后该project被变成其他project的依赖，而Podfile则为了编译当前project。需要注意的一点是ProjectName.podspec与Podfile两个文件中dependencies部分应该保持一致。

* #### 引用方式
其他工程如何引用【或者说依赖】私有的类库呢？一种方法是直接建一个私有specs repo, 并将添加对应spec。这种方法适合那么比较稳定的类库，不然会频繁地修改repo里对应的spec【每次添加版本号或修改commit】，对开发而言也不便利。另一种方式是直接再引用的Podfile里使用 ``` 
Pod 'customLib' , :git => 'https://xxxx' :commit -> 'yyyyy' 
```
的方式， 这种方法首先可以避免新建specs repo，不用每次去修改spec；其次在开发时，如果有修改依赖库，只需要将git地址指向本地地址就好,但这种方法需保证有spec文件在依赖库的代码根目录下。

### 阅读资料
* [cocoapods官网](http://cocoapods.org/)
* [使用CocoaPods来做iOS程序的包依赖管理](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)
* [CocoaPods进阶：本地包管理](http://www.iwangke.me/2013/04/18/advanced-cocoapods/)
