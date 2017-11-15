<!-- toc -->

# Memcache监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Memcache的数据采集可以通过采集脚本[memcached-monitor](https://github.com/iambocai/falcon-monit-scripts/tree/master/memcached)来做。

## 工作原理

memcached-monitor是一个cron，每分钟跑一次采集脚本```memcached-monitor.py```，脚本可以自动检测Memcached的端口，并连到Memcached实例，采集一些监控指标，比如get_hit_ratio、usage等等，然后组装为open-falcon规定的格式的数据，post给本机的falcon-agent。falcon-agent提供了一个http接口，使用方法可以参考[数据采集](../philosophy/data-collect.md)中的例子。

比如，我们有1000台机器都部署了Memcached实例，可以在这1000台机器上分别部署1000个cron，即：与Memcached实例一一对应。

需要说明的是，脚本```memcached-monitor.py```通过```ps -ef |grep memcached|grep -v grep |sed -n 's/.* *-p *\([0-9]\{1,5\}\).*/\1/p```来自动发现Memcached端口的。如果Memcached启动时 没有通过 ```-p```参数来指定端口，端口的自动发现将失败，这时需要手动修改脚本、指定端口。