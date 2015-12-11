#HAProxy 监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

HAProxy的数据采集可以通过[haproxymon](https://github.com/iask/haproxymon)来做。

## 工作原理

haproxymon是一个cron，每分钟跑一次采集脚本```haproxymon.py```，haproxymon通过Haproxy的stats socket接口来采集Haproxy基础状态信息，比如qcur、scur、rate等，然后组装为open-falcon规定的格式的数据，post给本机的falcon-agent。falcon-agent提供了一个http接口，使用方法可以参考[数据采集](../philosophy/data-collect.md)中的例子。