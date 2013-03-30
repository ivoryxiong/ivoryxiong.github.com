---
layout: post
title: "ios开发小结之屏幕亮度调节"
description: "在现在的阅读应用中，一般都有调节屏幕亮度的功能，但这个功能必要么，或者说实现难度如何呢？"
category: devops
tags:
    - software engineering
    - ios
---
{% include JB/setup %}

   在现在的阅读应用中，一般都有调节屏幕亮度的功能，但这个功能必要么，或者说实现难度如何呢？
   
   屏幕亮度调节的功能对于Andriod手机及安装了SBSetting类似软件的越狱iphone来说完全没有必要，这些设备上都有调节屏幕亮度的widget，只需要在屏幕上轻滑下拉便可方便地调节屏幕亮度（当然这个是全局的亮度）。但对于未越狱iphone而言，调节屏幕亮度意味着以下操作：按home健 -> 打开设置 -> 向上滚动到“亮度与墙纸”并选中进入新一页 -> 拖动silder调节。步骤多了而且需要退出应用（健忘者可能调节完后都忘记了刚才在使用什么app），故在应用内做到调节屏幕亮度比较重要。
  
   在ios5.0之前，苹果不提供屏幕亮度调节API，很不理解为什么这么设计，难道是怕开发者乱设置导致耗电量大，减短了续航时间么？ios5.0之后允许通过UIScreen的brightness属性调节，但这一设计却也不太合理（感觉Apple是真不乐意提供这么个功能…），比如这一属性只能在MainScreen上设置，所以设置这是全局的，不可随App而改变，另外这个设置是暂时的，手机如果进入待机状态后再打开，之前的设置无效。所以在Apple某些stupid的idea或design下，一个本来几行代码就搞定的事情就变成复杂了…
   
   如果想使用SDK提供的brighness，那么就在进入App的时候在设置,在退出App的时候保存当前的屏幕亮度（确保手机待机后返回App时设置还有效），如下：
{% highlight objc linenos %}
- (void)applicationDidEnterBackground:(UIApplication *)application {
    [[NSUserDefaults standardUserDefaults] setFloat:[UIScreen mainScreen].brightness forKey:@"APP_BRIGHTNESS"];
}

- (void)applicationWillEnterForeground:(UIApplication *)application {
    [UIScreen mainScreen].brightness = [[NSUserDefaults standardUserDefaults] floatForKey:@"APP_BRIGHTNESS"];
}
{% endhighlight %}


但如果你想兼容更多版本的设备或者说只想设置在App内有效，那么上面的方法便不适用了，首先那段代码只支持ios5以上，其次SDK有个BUG或者设计失误（见[stackoverflow](http://stackoverflow.com/questions/8316358/)）。传统的做法是将一个backgroundColor是blackColor的UIView的加在UIViewController的view上面，然后通过调节该view的alpha值来调节亮度,demo代码见：[Brightness@GITHUB](https://github.com/ivoryxiong/Brightness)