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
## 什么是storm ##
> 数据处理或者说数据挖掘是个热门的话题，对应的存储及分析工具现在都一把把的。在大数据处理方面，基于hadoop衍生许多的分析工具，在很多应用领域如日志分析、模型训练、离线查询等，这些工具已经能满足大部分的需求。但基于hadoop的工具都会受限于hadoop的短板：适合批处理而非实时、流式处理。storm就是适用于流式计算的工具，弥补hadoop的不足是storm的设计初衷。
>
>流式计算提得比较多的领域是计算广告（yahoo!因此还自己整了个S4），特别适合处理广告的点击流。在互联网的在线广告系统中，展示策略和点击流处理是两个最重要的部分，其中点击流处理包括实时扣费、反作弊、在线统计等多个步骤。下面的例子就是基于点击流处理中的反作弊来试用下storm。

## 小试身手 ##
>   Topology、Spout、Bolt是Storm里的重要概念，流式计算让人容易想到水，所以数据源就用Spout来表示再也形象不过了，而计算过程可以理解为送自来水到用户家，自来水要经过的很多闸门Bolt对应可以理解为完整计算中的各个步骤。而Topology就对应Hadoop里的job，更形象地描述了处理过程（拓扑结构）:
>   ![topology] (https://raw.github.com/wiki/nathanmarz/storm/images/topology.png)
>
>   如图，Topology由多个Spout和Bolt组成，Spout也可理解为没有输入的Bolt，一个Topology不会因为Spout输出结束而结束，必须主动去暂停。废话少说，上例子。
###定义Topology###
{% highlight java linenos %}
public class ClickStreamTopology {

    public static void main(String[] args) {
        try {
            TopologyBuilder builder = new TopologyBuilder();
            builder.setSpout("click-data", new UserClickSpout());
            builder.setBolt("anti-spam", new AntiSpamBolt()).shuffleGrouping("click-data");
            builder.setBolt("stat", new StatBolt()).shuffleGrouping("anti-spam");

            Config conf = new Config();
            conf.setMaxTaskParallelism(3);
            conf.setDebug(true);
            conf.put("click", args[0]);

            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology("click-stream", conf, builder.createTopology());
            Utils.sleep(30000);
            cluster.killTopology("click-stream");
            cluster.shutdown();
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
}
{% endhighlight %}
>如代码所示，Topology由于需要很多配置，所以采用了Builder模式来构建Topology实例，6-8行分别设置了Spout和两个Bolt，并且后面一个Bolt(Spout是特殊的Bolt)的输入是前一个Bolt的输出。 10-13行对整个Topology的执行环境进行了配置，15-16行获得可执行topology的storm集群并将topology提交。17-20行代码在topology执行30s后主动终止topology并关闭集群。
### 实现Spout和Bolt###
> * spout
> UserClicjSpout继承BaseRichSpout，并在open方法里打开输入源（这里是一个文本文件），然后在nextTuple方法里一行一行地读入并按照declareOutputFields方法定义的格式转换。
{% highlight java linenos%}
public class UserClickSpout extends BaseRichSpout {
    ...    
    @Override
    public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
        this.collector = collector;
        String clickFile = (String) conf.get("click");
        try {
            scanner = new Scanner(new FileInputStream(clickFile));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void nextTuple() {
        Utils.sleep(10);
        if (scanner != null && scanner.hasNext()) {
            String line = scanner.nextLine().trim();
            String [] tmps = line.split(" ");
            System.out.println(tmps.length);
            if (tmps.length == 4 ) {
                Values val = new Values();
                for (String str: tmps) {
                    val.add(str);
                }
                collector.emit(val);
            } else {
                collector.emit(new Values("","","",""));
            }
        } else if (scanner != null) {
            scanner.close();
            scanner = null;
        }
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("time", "url", "referer", "ip"));
    }
}
{% endhighlight %}
> * Bolt
> AntiSpamBolt从UserClickSpout读入数据（time,url，referer,ip的四元组），并在execute方法里进行逻辑处理，处理完后按照declareOutputFields定义的格式将结果传递给下一个Bolt(这里是指StatBolt)。当然，Bolt链中的某个Bolt也可以将处理完后的数据往后传。AntiSpamBolt中有多个Filter,按照实际生产环境也可以将Filter拆分成不同的Bolt，这样就更能体现流式计算的思想(*^__^*) 
{% highlight java linenos %}
public class AntiSpamBolt extends BaseRichBolt {
    private static List<Filter> filters ;
    private OutputCollector collector;
    
    static {
        filters = new ArrayList<>();
        filters.add(new IpBlackListFilter());
    }

    public void prepare(Map stormConf, TopologyContext context, OutputCollector collector) {
        this.collector = collector;
    }

    public void execute(Tuple input) {
        Values values = new Values(input.getValues().toArray());
        for(Filter filter: filters) {
            if (filter.filter(input)) {
                values.add(true);
            }
        }
        values.add(false);
        collector.emit(values);
        collector.ack(input);
    }

    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("time", "url2", "referer", "ip", "spam"));
    }
}
{% endhighlight%}

更详细的就不讲了，具体代码见[tutorial-storm](https://github.com/ivoryxiong/tutorial/tree/master/tutorial-storm),也可以从[storm-wiki](https://github.com/nathanmarz/storm/wiki)理解更详细的内容。
