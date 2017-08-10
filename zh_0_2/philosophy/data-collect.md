作为监控系统来讲，首先得有监控数据，然后才能做后面的分析处理、绘图报警等事情，那falcon是如何处理数据采集这个问题的呢？

我们先要考虑有哪些数据要采集，脑洞打开~

- 机器负载信息，这个最常见，cpu.idle/load.1min/mem.memfree.percent/df.bytes.free.percent等等
- 硬件信息，比如功耗、风扇转速、磁盘是否可写，系统组同学对这些比较关注
- 服务监控数据，比如某个接口每分钟调用的次数，latency等等
- 数据库、HBase、Redis、Openstack等开源软件的监控指标

要采集的数据还挺多哩，监控系统的开发人员不是神，没法搞定所有数据，比如MySQL，DBA最懂，他知道应该采集哪些指标，监控只要提供一个数据push的接口即可，大家共建。想知道push给Server的数据长啥样？可以参考[Tag与HostGroup设计理念](tags-and-hostgroup.md)中提到的两条json数据

上面四个方面比较有代表性，咱们挨个阐述。

**机器负载信息**

这部分比较通用，我们提供了一个agent部署在所有机器上去采集。不像zabbix，要采集什么数据需要在服务端配置，falcon无需配置，只要agent部署到机器上，配置好heartbeat和Transfer地址，就自动开始采集了，省去了用户配置的麻烦。目前agent只支持64位Linux，Mac、Windows均不支持。

**硬件信息**

硬件信息的采集脚本由系统组同学提供，作为plugin依托于agent运行，plugin机制介绍请看[这里](plugin.md)。

**服务监控数据**

服务的监控指标采集脚本，通常都是跟着服务的code走的，服务上线或者扩容，这个脚本也跟着上线或者扩容，服务下线，这个采集脚本也要相应下线。公司里Java的项目有不少，研发那边就提供了一个通用jar包，只要引入这个jar包，就可以自动采集接口的调用次数、延迟时间等数据。然后将采集到的数据push给监控，一分钟push一次。目前falcon的agent提供了一个简单的http接口，这个jar包采集到数据之后是post给本机agent。向agent推送数据的一个简单例子，如下：

```bash
curl -X POST -d '[{"metric": "qps", "endpoint": "open-falcon-graph01.bj", "timestamp": 1431347802, "step": 60,"value": 9,"counterType": "GAUGE","tags": "project=falcon,module=graph"}]' http://127.0.0.1:1988/v1/push
```

**各种开源软件的监控指标**

这都是大用户，比如DBA自己写一些采集脚本，连到各个MySQL实例上去采集数据，完事直接调用server端的jsonrpc汇报数据，一分钟一次，每次甚至push几十万条数据，比较好的发送方式是500条数据做一个batch，别几十万数据一次性发送。

