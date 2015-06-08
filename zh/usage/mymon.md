# MySQL监控实践

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

MySQL的数据采集可以通过[mymon](https://github.com/open-falcon/mymon)来做。

## 工作原理

mymon是一个cron，每分钟跑一次，配置文件中配置了数据库连接地址，mymon连到该数据库，采集一些监控指标，比如global status, global variables, slave status等等，然后组装为open-falcon规定的格式的数据，post给本机的falcon-agent。falcon-agent提供了一个http接口，使用方法可以参考[数据采集](../philosophy/data-collect.md)中的例子。

比如我们有1000台机器都部署了MySQL实例，可以在这1000台机器上分别部署1000个cron，即：与数据库实例一一对应。

## 代码

代码在这里：https://github.com/open-falcon/mymon
