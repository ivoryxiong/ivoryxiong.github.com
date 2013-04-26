---
layout: page
title: home
tags:
    - 技术
    - ios
    - java
    - 编程
    - 开发
    - 计算广告
    - 算法
    - 系统架构
description: "blog from ivoryxiong"

---
{% include JB/setup %}
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
