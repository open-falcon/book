<!-- toc -->

# Solr监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Solr的数据采集可以通过脚本[solr_monitor](https://github.com/shanshouchen/falcon-scripts/tree/master/solr-monitor)来做。

## 工作原理

solr_monitor是一个cron，每分钟跑一次脚本```solr_monitor.py```，主要采集一些solr实例内存信息和缓存命中信息等等，然后组装为open-falcon规定的格式的数据，post给本机的falcon-agent。

脚本可以部署到Solr的各个实例，每个实例上运行一个cron，定时执行数据收集，即：与Solr实例一一对应

如果一台服务器存在多个Solr实例，可以通过修改```solr_monitor.py```中的```servers```属性，增加Solr实例的地址完成本地一对多的数据收集
