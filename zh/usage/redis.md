# Redis监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Redis的数据采集可以通过采集脚本[redis-monitor](https://github.com/iambocai/falcon-monit-scripts/tree/master/redis)来做。

## 工作原理

redis-monitor是一个cron，每分钟跑一次采集脚本```redis-monitor.py```，其中配置了redis服务的地址，redis-monitor连到redis实例，采集一些监控指标，比如connected_clients、used_memory等等，然后组装为open-falcon规定的格式的数据，post给本机的falcon-agent。falcon-agent提供了一个http接口，使用方法可以参考[数据采集](../philosophy/data-collect.md)中的例子。

比如，我们有1000台机器都部署了Redis实例，可以在这1000台机器上分别部署1000个cron，即：与Redis实例一一对应。
