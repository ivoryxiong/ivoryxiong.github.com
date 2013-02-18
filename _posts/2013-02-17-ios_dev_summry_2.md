---
layout: post
title: "ios开发小结之app发布升级"
description: "记录并总结下最近2个月ios开发中关于app审核及分布遇到的问题及想法"
category: devops
tags: 
    - software engineering
    - ios
    - summry
---
{% include JB/setup %}
在近两个月的开发中，遇到了挺多问题的，几天加班加点，最后还是在年前发布并更新了一个版本，欢迎下载[无觅下载](https://itunes.apple.com/us/app/wu-mi-yue-du-zhi-neng-cai/id593518746?ls=1&mt=8)。

最头疼的问题是提交app审核，之前的工程不太规范，导致一些文件icon没有设置好，直接validate就失败了几次。validate通过后便upload，但upload过程中又出错了，原因是java版本不对（xcode使用java类库进行上传），解决方法在见[stackoverflow上的回答](http://stackoverflow.com/questions/10011635/an-error-occurred-uploading-to-the-itunes-store-please-upgrade-java)。一切搞定后，还得对付国内的龟速网络，一个15M不到的ipa竟然上传了快半个小时，算下来速度连10Kb/s都不到（带宽可是20M的呀！），聊着天把ipa上传给app store都快晚上2点了，坑爹呀。再说说关于app审核发布的问题，“无觅阅读”因为metadata被rejected一次，原因在于我们的描述里出现了安卓这个词，话说apple这个政策也真是小家子气，这么封闭可别最后落个固步自封的境地呀。新app审核的时间大概在2周左右，主要是排队的时间长，花费了一周多的时间，真正审核的时间也就2-3天，更新app的审核周期更短些，我们两个版本都刚好花费1周的时间。另外需要注意的在创建版本时要选择“Hold for Developer Release ”，避免由于时差，app半夜就发布出去了，而导致运营和推广安排滞后。

无觅是小团队，测试方面人力不足，做得不是很好，app推出后没有少挨用户骂。不断迭代，提高app质量是发布后开发人员的重要工作，而收集crash log是则是fix bug的第一步。我们在app中同时使用了umeng和GA两种统计工具，同时umeng还兼有收集crash log的用处。但umeng sdkv2.13之前的版本都存在Crash Log中dSYM UUID生成的bug，导致能看到错误日志，也无法将错误对应到具体代码。使用了umeng的分析工具，错误分析变得简单，但分析的基本原理还是要懂些（全是编译原理的东东），可以看看下面这些资料：
> 1. [连接crash log的符号表](http://stackoverflow.com/questions/6248132/symbolicate-xcode4-crash-reports/6291663)
> 2. [mac下反汇编工具](http://www.verydemo.com/demo_c134_i2411.html)
> 3. [连接符号表的原理](http://stackoverflow.com/questions/13574933/ios-crash-reports-atos-not-working-as-expected)（这个刚好是umeng sdkv2.1.3之前版本的bug）

上面这点资料很少，但比较有用。从umeng分析工具和资料3知道，每次发布新版本后，都需要将archive和当前版本的代码保存下来以便日后分析crash log。

对于国内开发者而言，除了提交app给app store，还可以提交给很多的越狱市场（目前国内知道的有5、6个），但一个个手动打包很麻烦，所以写了个脚本来做这件事情。
{% highlight bash linenos %}
#!/bin/sh
on_fail() {
    echo "$*" >&2
    exit 1
}

build() {
    echo "using configuration ${CONFIG}"
    echo "version suffix:${VERSION}"
    echo "cleaning build folder:${BUILD_FOLDER}"
    rm -rf "${BUILD_FOLDER}" || die "error when cleaning build folder"
    echo "cleaning build done"
    xcodebuild -configuration "${CONFIG}" build DISPATCHER_CHANNEL=${DISPATCHER_CHANNEL} BUNDLE_VERSION=${VERSION} || on_fail "build failed"
}


package() {
    echo "making ipa…"
    IPA_NAME="${APPLICATION_NAME}_${VERSION}_${DISPATCHER_CHANNEL}.ipa"
    xcrun -sdk iphoneos PackageApplication -v "${BUILD_FOLDER}/${CONFIG}-iphoneos/${APPLICATION_NAME}.app" -o "${RELEASE_FOLDER}/${IPA_NAME}" --sign "${DEVELOPER_NAME}" --embed "${PROVISONING_PROFILE}" || on_fail "package failed!"
    echo "finished making ${IPA_NAME}"
}

usage() {
    echo "Usage: $0 version"
    exit 1
}

#setup
if [ $# -ne 1 ]; then
   usage
fi
PROJ_FOLDER="$(pwd)"
RELEASE_FOLDER="${PROJ_FOLDER}/ipa"
BUILD_FOLDER="${PROJ_FOLDER}/build"
APPLICATION_NAME='MyApp'
CONFIG='Release'
DEVELOPER_NAME="MyComp"
PROVISONING_PROFILE="config/production.mobileprovision"
VERSION=$1

if [ ! -d "$RELEASE_FOLDER" ]; then
    mkdir "$RELEASE_FOLDER"
fi
for line in $(cat config/stores.dat)
do
    DISPATCHER_CHANNEL="${line#*:}"
    build
    package
done
echo "build successfully!"
exit 0

{% endhighlight %}
执行上述脚本必须安装xcode的command line tools，商店渠道保存在config/stores.dat文件中，而开发者证书则为config/production.mobileprovision。
