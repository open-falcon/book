# 集群聚合

集群监控的本质是一个聚合功能。

单台机器的监控指标难以反应整个集群的情况，我们需要把整个集群的机器（体现为某个HostGroup下的机器）综合起来看。比如所有机器的qps加和才是整个集群的qps，所有机器的request_fail数量 ÷ 所有机器的request_total数量=整个集群的请求失败率。

我们计算出集群的某个整体指标之后，也会有“查看该指标的历史趋势图” “为该指标配置报警” 这种需求，故而，我们会把这个指标重新push回监控server端，于是，你就可以把它当成一个普通监控数据来对待了。

背景交代清楚了……现在来介绍我们的实现……

首先，用户要在某个HostGroup下去添加集群聚合规则，我们就知道这个规则涵盖的机器是当前这个HostGroup下的机器。

其次，整个集群的指标计算是一个除法，除法的话就有分子，有分母。上面提到的“所有机器的qps加和才是整个集群的qps” 这个场景中每个机器应该有个qps的counter，每个counter在书写的时候要求用$()包裹起来，故而，分子可以这么描述：`$(qps/module=judge,project=falcon)`，分母就填写 1 就行了。

对于上面所述场景二：“所有机器的request_fail数量 ÷ 所有机器的request_total数量=整个集群的请求失败率” 分子就是：`$(request_fail/module=graph,project=falcon)` 分母就是：`$(request_total/module=graph,project=falcon) `

另外，对于分子和分母，我们是支持加减计算的，不过不支持除法、乘法、括号，举个没意义的分母：`$(cpu.idle) + $(cpu.busy) `，可以两个counter相加

分母和分母不但支持配置counter，也支持配置成纯数字，支持配置 `$#`，`$#`在shell编程中是参数个数的意思，我们这里也类似。比如我们这样的配置：

```
xbox节点：cop.xiaomi_owt.inf_pdl.falcon_service.judge
分子：$(qps/module=judge,project=falcon)
分母：$#
```

对于分子而言，我们就会拿着HostGroup下的所有机器去query这个counter的最新值，然后把所有值相加（不同机器计算出来的分子要相加），但是有的机器可能查不到数据，`$#`表示的是正常查到数据的机器数量。整个表达式假设涉及到3个counter，对某个机器而言，必须3个counter都查到数据才被使用，只要有一个counter没有查到数据，那就忽略这个机器。
 
注意：分子、分母中至少配置一个counter，不能都配置成纯数字或$#，因为这种配置是没啥意义的。除了配置分子、分母之外，还有很多其他配置，比如endpoint、metric、tags、step等等，这些是为了把数据重新push回监控server的时候用的。监控的数据有好几个字段，缺一不可，我们可以计算出集群监控指标的value和时间戳，但是无法自动填充endpoint、metric、tags、step等字段，所以，仍然要用户手工填写。



##用户手册
使用集群聚合监控，需要进行两个配置: cluster配置 和 策略配置。下面，我们以一个例子，讲述如何使用集群聚合监控提供的服务。
###用户需求
获取节点cop.xiaomi_owt.inf_pdl.falcon_service.judge下所有机器的$(cpu.idle)-$(cpu.nice)-$(cpu.guest)的平均值，达到阈值后通知给用户。
###Cluster monitor配置
访问portal服务，搜索节点cop.xiaomi_owt.inf_pdl.falcon_service.judge，点击后面的aggregator链接，进入当前节点对应的aggregator列表，如下图所示：
![aggregator.config](https://raw.githubusercontent.com/yindashan/yindashan.common.store/master/images/open-falcon/aggregator/cluster1.jpg)

点击右上角的“新建”按钮，可进入aggregator编辑页面，如下图所示：
![aggregator.edit](https://raw.githubusercontent.com/yindashan/yindashan.common.store/master/images/open-falcon/aggregator/cluster2.jpg)

###策略配置
配置了cluster monitor后，如果想收到关于聚合的值的报警，需要在节点绑定的模板里面配下监控策略：
![aggregator.edit](https://raw.githubusercontent.com/yindashan/yindashan.common.store/master/images/open-falcon/aggregator/cluster3.png)


