---
layout: post
title: "iOS开发总结之Https&SSL Pinning"
description: "互联网发展到今天，早已不是十多年前的莽荒时代，用户越来越关注信息的安全性，曾经野蛮生长的那些互联网产品，在黑暗世界的冰山一角暴露给公众后，受到用户不断的抨击。作为一名合格开发者，确保用户信息安全已经是共识，本文简单介绍经常使用的Https协议及在iOS系统上的一些实现"
category: devops 
tags:
    - ios
    - https
    - ssl pinning

---

互联网发展到今天，早已不是十多年前的莽荒时代，用户越来越关注信息的安全性，曾经野蛮生长的那些互联网产品，在黑暗世界的冰山一角暴露给公众后，受到用户不断的抨击。作为一名合格开发者，确保用户信息安全已经是共识，本文简单介绍经常使用的Https协议及在iOS系统上的一些实现。

### Https协议 ###
https协议全称超文本传输安全协议，是超文本传输协议和SSL/TLS的组合，用以提供加密通讯及对网络服务器身份的鉴定。与Http的区别在于，Https多了对通信内容的加密及对服务器身份鉴定。Client端与Server端通信的过程大致如下图：
![https协议](/images/Https.png)

里面步骤的具体解释参见博文[图解HTTPS](https://kb.cnblogs.com/page/112881/)，本文对其中几个部分详细解释下：

1. Server在发给Client的证书中，包含了Server证书到根证书颁发机构的所有层级【一般有3、4个，见gmail如下图】，而Client在接受到后会对照自己知道的CA列表【一般的通用程序如浏览器、JRE，都只包含[根证书颁发机构列表](https://www.mozilla.org/en-US/about/governance/policies/security-group/certs/included/)】逐个验证,具体的验证逻辑可能会因为Client的不同而有所区别，比如iOS中AFNetworking就比较简单，直接比较SecKeyRef对象。

    ![gmail ca](/images/gmail_ca.png)
2. Https协议并没有规定双方使用何种加密方法，加密方法由Server证书指定。在给Client的CA证书中，公共密钥项中给出了加密方法【具体支持的加密方法见RFC1423】与Public Key, 如下图，Gmail使用了安全性比较高的椭圆曲线算法。

    ![gamil ca public key](/images/gmail_public_key.png)


### SSL Pinning ###

Https对比Http已经很安全，但在建立安全链接的过程中，可能遭受[中间人攻击](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)。防御这种类型攻击的最直接方式是Client使用者能正确鉴定Server发的证书【目前很多浏览器在这方面做的足够好，用户只要不在遇到警告时还继续其中的危险操作】，而对于Client的开发者而言，一种方式保持一个可信的根证书颁发机构列表，确认可信的证书，警告或阻止不是可信根证书颁发机构颁发的证书。

SSL Pinning其实就是证书绑定，一般浏览器的做法是信任可信根证书颁发机构颁发的证书，但在移动端【非浏览器的桌面应用亦如此】，应用只和少数的几个Server有交互，所以可以做得更极致点，直接就在应用内保留需要使用的具体Server的证书。对于iOS开发者而言，如果使用AFNetwoking作为网络库，那么要做到这点就很方便，直接证书作为资源打包进去就好，AFNetworking会自动加载，具体代码就不贴了，nsscreencast已经有很好的[tutorial](https://nsscreencast.com/episodes/73-ssl-pinning)。

### 阅读资料 ###
* [SSL Pinning](https://nsscreencast.com/episodes/73-ssl-pinning)
* [图解HTTPS](https://kb.cnblogs.com/page/112881/)
* [根证书颁发机构](https://en.wikipedia.org/wiki/Root_certificate)
* [HTTPS那些事（一）HTTPS原理](https://www.guokr.com/post/114121/)
* [The First Few Milliseconds of an HTTPS Connection](https://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)
* [RFC2616: Hypertext Transfer Protocol -- HTTP/1.1](https://www.ietf.org/rfc/rfc2616)
* [RFC1422: X.509 Architecture - Certificate Definition](https://www.ietf.org/rfc/rfc1422)
* [RFC1423: Algorithms, Modes, and Identifiers](https://www.ietf.org/rfc/rfc1423)
