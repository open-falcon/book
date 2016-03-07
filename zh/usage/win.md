# 监控Windows平台

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Windows主机的运行指标的采集，可以写Python脚本，通过windows的计划任务来每分钟执行采集各项运行指标，包括内存占用、CPU使用、磁盘使用量、网卡流量等。

可以直接使用 [windows_collect脚本](https://github.com/freedomkk-qfeng/falcon-scripts/tree/master/windows_collect) 来实现对windows主机的监控指标采集。


## 使用方式

- 根据实际部署情况，修改脚本开头的配置参数
- 修改 graph 的 mysql 编码为utf8，以支持中文的。由于 windows 的网卡有可能存在中文，所以这一步很重要。。。
- 测试： python windows_collect.py
- 丢进 windows 计划任务完事

## 已经测试过的环境有

- windows 10
- windows 7
- windows server 2012


------

或者也可以直接使用golang版本windows agent： https://github.com/LeonZYang/agent
