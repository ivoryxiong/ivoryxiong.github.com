---
layout: post
title: "在jekyll里使用无觅相关文章插件"
description: "详细介绍如何在jekyll里使用无觅相关文章插件"
category: tutorial
tags:
    - jekyll
    - wumii
---
继wordpress后，jekyll + github成为越来越多人的写作方式。jekyll能取代wordpress可能因为这几个原因：

1. 成本更低，使用wordpress需要有功能齐全的空间和域名，而由于github page提供了对jekyll的支持，只需搞一个域名。
2. 功能插件丰富，创作者使用静态网页也能搭建一个类似wordpress这样功能齐全的blog。比如评论插件有disqus、友言，统计有GA，登陆有灯鹭，相关文章有无觅...
3. 使用git这样的scm，使得创作长篇文章更方便。

这几年写blog一直断断续续，除了自己比较懒之外，找不到靠谱的wordpress托管也是一个大问题，很早开始写流水blog，开始在chinaunix写，后来在bloghost买的空间和域名，过了半年多说要备案...因此让bloghost换到USA的机房，结果没换成（吐槽下bloghost）,最后不了了之。年初，工作半年多了，感觉自我提升很慢，越发感觉到看blog和写blog的重要性，于是在hostgator上买了个baby plan，但wordpress上写的不是很惬意，特别是排版问题。最后看了[阳志平的blog](http://www.yangzhiping.com/tech/wordpress-to-jekyll.html),果断放弃空间，携域名转向jekyll与github。
中文blog从wordpress里转向jekyll可能会遇到jekyll没有很好的相关文章插件问题，但其实国内目前最好的相关文章插件：[无觅相关文章插件](http://app.wumii.com/widget/relatedItems)是可以很方便地继承到jekyll里的。具体步骤如下：

step1. 使用[jekyll-bootstrap](http://jekyllbootstrap.com/)安装好jekyll

step2. 从[无觅相关文章]（http://app.wumii.com/site/index）注册并验证站点

step3. 将文件_includes/themes/twitter/post.html（使用不同的皮肤文件路径不一样）的(\为转义，需去掉)

``` javascript
{\% include JB/comments \%}
```

替换成

``` javascript
{\% include JB/related-posts \%}
{\% include JB/comments \%}
```
step4. 创建文件 _include/JB/related-posts,内容为

``` javascript
<script type="text/javascript" id="wumiiRelatedItems"></script>
<script type="text/javascript">
var wumiiPermaLink = "http://ivoryxiong.org{{page.url}}";
var wumiiTitle = "{{page.title}}";
var wumiiTags = " {% assign first_tag = nil %}{% for tag in page.tags %}{% if first_tag != null %},{% endif %} {% assign first_tag = tag %} {{tag}} {% endfor %}";
var wumiiSitePrefix = "http://ivoryxiong.org/";
var wumiiParams = "&num=5&mode=3&pf=JAVASCRIPT";
</script>
<script type="text/javascript" src="http://widget.wumii.com/ext/relatedItemsWidget"></script>
<a href="http://www.wumii.com/widget/relatedItems" style="border:0;">
<img src="http://static.wumii.cn/images/pixel.png" alt="wumii related posts" style="border:0;padding:0;margin:0;" />
</a>
```

>代码中http://ivoryxiong.org需要替换成自己的域名

step5. push到github，完工。

jekyll和github都是好东西，希望更多人会喜欢，效果便如本页,截图就不上了，。
