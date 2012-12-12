---
layout: post
title: "ios开发小结"
description: "接触ios开发有段时间，趁有时间整理下关于代码质量及一些功能开发的感想。"
category: devops
tags: 
    - software engineering
    - ios
---
{% include JB/setup %}
## ios开发总结 ##

最近在接手外包公司做的iphone终端，并对外包的代码做了一些调整和改进。在这段时间的开发过程中有一些感想及总结，写下来做个备忘。

### 代码质量 ###
外包交付的代码质量不是很好，对代码进行统计就会发现问题：
{% highlight bash %}
find . -name \*.m -print | grep "WM" | xargs wc -l | sort -nk 1 | tail -n 50 | head -n 49 | awk '{printf("%s **/*%s\n",$1,substr($2,length($2)-9,10));}'
{% endhighlight %}

统计结果如下图：

![代码统计](/imgs/code_stat.png)
从统计结果看，有很多代码实现的文件行数已经超过200行，甚至快到1000行。这样的代码可读性很差，可以反应出的问题是不够代码逻辑不够抽象，复用机制使用得少。

另外一个问题是，外观布局太依赖于InterfaceBuilder，这导致一些通用部件（比如navigation上的button）会被频繁地引入到xib文件中，一旦这些部件要做调整，那么就坑爹了（我被坑了o(╯□╰)o），必须一个个调！

app代码质量的提高，可以从下面两方面着手：

#### 控制代码行数 ####

有意识地控制每个文件的代码行数能迫使自己去改进程序的结构以达到复用代码的效果。一个源代码文件多少行会比较合适呢？在stackoverflow上有几个相似的问题:
	
[Good practice class line count](http://stackoverflow.com/questions/1086851/good-practice-class-line-count/1086865)
	
[how-many-lines-of-code-is-too-many](http://stackoverflow.com/questions/20981/how-many-lines-of-code-is-too-many)
	
其中有两个答案比较有道理：

   * 为了阅读方便，一屏
   * 不多于500行，小于300行最好，多于500行就需要重构了
	
而缩小源文件最好的方法便是利用重构技术里的抽取函数、拆分功能、构造类继承体系。详细方法参看[重构:改善既有代码的设计](http://www.amazon.cn/dp/B003BY6PLK/)一书。

#### 减少使用IB ####
* 公用组件单独使用IB构建，以代码形式包含在视图基类中，而非包含在主要视图中，避免不断的copy。
* 由于iphone5机体长度不一样，需要适配，如使用IB构建，一般需要为iPhone5单独构建一套，如使用代码，则可通过计算去适配（除非应用本来在两个设备上就表现不同）。

	
### 一些功能 ###

1. 推送通知

ios的消息推送因为APNS的存在而简化了开发流程，如果不喜欢APNS，页可以使用第三方提供的服务[极光推送](http://www.jpush.cn/index.jsp)。
	
推送的原理都一样，App将设备的token（该token应该由APNS或第三方服务提供）上传到引用服务器，然后有在需要推送消息的，将消息按照一定格式组织，并携带上token发送给推送服务APNS（第三方服务），然后第三方服务根据token向设备推送。

2. 统计分析

统计分析服务很多，比较常用的包块GA和友盟，功能都差不多，但考虑到GFW，比较倾向于友盟。
	
申请好账号后就是将统计代码潜入到app中了，在ios app中比较好的嵌入方式有两种：
	* 利用objc的category特性扩展UIViewController，添加统计的功能。
	* 利用类继承体系，给所有自定义的ViewController添加一个父类，并覆盖其中viewWillAppear和viewWillDisapper方法。

3. 自动更新

app的自动更新基本上标配，不同于Android，ios的可以直接利用UIApplication单例直接跳转到App Store下载，而且越狱的设备也可以访问，十分方便。
	
4. 第三方登录

移动应用越来越专注于主要功能而弱化其他辅助功能，而且为了利用社交网络的社会化影响，账号系统一般都提供第三方登录。就App而言，体验最好的是利用SINA的SSO登陆，能减少用户输入，提供体验。不过SINA的SSO的ISO SDK在设计上有一点小瑕疵：需要保存app key等敏感信息在客户端，其中的隐患可以参见文章[app反编译及敏感信息存储](http://ivoryxiong.org/tutorial/2012/11/16/app-decompile-info-store/)。

写得很凌乱，权当备忘。