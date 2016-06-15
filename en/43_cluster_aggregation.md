##Cluster aggregation


The nature of cluster monitor is an aggregation function. 

The monitor indexes of a single machine can not reflect the situation of a whole cluster, so we need to consider the machines  in the whole cluster (reflected as the machines in certain HostGroup) as a group. For instance, the sum of qps for all machines is the qps for the cluster, the number of request_fail for all machines divided by the number of request_total for all machines equals the request failure rate of the whole cluster.

When we calculated total index for the cluster, there will be the requirements as “check the history tendency chart of the index” “configure alarm for the index”. Therefore, we will again push the index back to monitor server, then you can treat it as an ordinary monitor data. 

The background is clear...... Now introduce how to realize...... 

First, users need to add the cluster aggregation rules under certain HostGroup, and we know the machines under the rules are under the HostGroup. 

Second, the index calculation of the whole cluster is a division. There is a numerator and a denominator in a division. The sentence of “the sum of qps for all machines is the qps for the cluster” was mentioned in above. In this situation, every machine should have a counter for qps, and every counter needs to be covered with $() in writing. Therefore, the numerator can be described as: ```$(qps/module=judge,project=falcon)```, and denominator as 1. 

For the other situation described above as: “the number of request_fail for all machines divided by the number of request_total for all machines equals the request failure rate of the whole cluster”, the numerator is: ```$(request_fail/module=graph,project=falcon)```, the denominator is: ```$(request_total/module=graph,project=falcon)```.

Besides, for numerator and denominator, we support addition and subtraction, and do not support multiplication, division and brackets. For example a meaningless denominator:```$(cpu.idle) + $(cpu.busy)```, 2 counters can be added. 

The denominator and another not only support to configure counter, but also support to configure pure numbers, and configure $#.  $# in shell programming means the number of parameters, and here we mean the same. For instance we have the configuration such as: 

```
xbox node: cop.xiaomi_owt.inf_pdl.falcon_service.judge
Numerator: $(qps/module=judge,project=falcon)
Denominator: $#
```

For numerator, we will use all the machines under HostGroup to query the newest value of counter, and add all values together (numerators calculated from different machines need to be added together). But the data of some machines possibly can not be found. $# indicates the number of machines can be found normally. The whole expression hypothetically involves 3 counters. For one certain machine, it can be used only if the data of its 3 counters can be all found. We ignore the machine if any data of one counter can not be found. 

Notice: As least configure one counter for numerator and denominator, and they can not be all configured as pure numbers or $#, because it is meaningless. Besides the configuration for numerator and denominator, there are many other configurations such as endpoint, metric, tags, steps, etc., which is used for the time of pushing data back to monitor server. There are several fields in monitor data which are indispensable. We can calculate the value and time for cluster monitor indexes, but cannot automatically fill in the fields as endpoint, metric, tags, steps, etc. Therefore, they need to be filled manually by users. 

##User manual

To use cluster aggregation monitor, we need two configurations: cluster configuration and strategy configuration. Now we will introduce how to use services provided by cluster aggregation monitor with an example.

##User requirements

We obtain the average value $(cpu.idle)-$(cpu.nice)-$(cpu.guest) of all machines under the node named cop.xiaomi_owt.inf_pdl.falcon_service.judge, and inform users when reaches the threshold value.

##Cluster monitor configuration
Visit portal service, search the node cop.xiaomi_owt.inf_pdl.falcon_service.judge, click the aggregator link afterwords, and enter the corresponding aggregator list for current node, the procedure is shown as follows:  

##picture

Click the “create” button on the right top corner, and enter the aggregator edit page, which is shown in the following picture: 

##picture

##Strategy configuration

After configured with cluster monitor, if we want to receive alarm about the value of aggregation, we need to add monitor strategy in the template bound to the node:  

#picture