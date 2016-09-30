# 监控Windows平台

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Windows主机的运行指标的采集，可以写Python脚本，通过windows的计划任务来每分钟执行采集各项运行指标，包括内存占用、CPU使用、磁盘使用量、网卡流量等。

可以直接使用以下 window 监控程序进行 windows 主机的监控指标采集。

- [windows_collect](https://github.com/freedomkk-qfeng/falcon-scripts/tree/master/windows_collect)：python脚本
- [windows-agent](https://github.com/LeonZYang/agent)： go 语言实现的 agent
- [Windows-Agent](https://github.com/AutohomeRadar/Windows-Agent)：汽车之家开源的作为Windows Service运行的Agent，python实现。
- [windows-agent](https://github.com/freedomkk-qfeng/windows-agent)：另一个 go 语言实现的 windows-agent。支持端口，进程监控，支持后台服务运行。

