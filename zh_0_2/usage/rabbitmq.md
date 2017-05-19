# RMQ监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

RMQ的数据采集可以通过脚本[rabbitmq-monitor](https://github.com/iambocai/falcon-monit-scripts/tree/master/rabbitmq)来做。

## 工作原理

rabbitmq-monitor是一个cron，每分钟跑一次脚本```rabbitmq-monitor.py```，其中配置了RMQ的用户名&密码等，脚本连到该RMQ实例，采集一些监控指标，比如messages_ready、messages_total、deliver_rate、publish_rate等等，然后组装为open-falcon规定的格式的数据，post给本机的falcon-agent。falcon-agent提供了一个http接口，使用方法可以参考[数据采集](../philosophy/data-collect.md)中的例子。

比如我们部署了5个RMQ实例，可以在 每个RMQ实例机器上运行一个cron，即：与RMQ实例一一对应。
