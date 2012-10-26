---
layout: post
title: "A Simple Tutorial for Storm"
description: "a simple program user storm to process click stream"
category: tutorial 
tags: 
    - storm
    - stream computing
    - data processing 
---
{% include JB/setup %}
### 什么是storm ###
> 数据处理或者说数据挖掘是个热门的话题，对应的存储及分析工具现在都一把把的。在大数据处理方面，基于hadoop衍生许多的分析工具，在很多应用领域如日志分析、模型训练、离线查询等，这些工具已经能满足大部分的需求。但基于hadoop的工具都会受限于hadoop的短板：适合批处理而非实时、流式处理。storm就是适用于流式计算的工具，弥补hadoop的不足是storm的设计初衷。
>
>流式计算提得比较多的领域是计算广告（yahoo!因此还自己整了个S4），特别适合处理广告的点击流。在互联网的在线广告系统中，展示策略和点击流处理是两个最重要的部分，其中点击流处理包括实时扣费、反作弊、在线统计等多个步骤。下面的例子就是基于点击流处理中的反作弊来试用下storm。

### 小试身手 ###

