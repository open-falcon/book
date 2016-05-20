
# Docker容器监控实践

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

docker container的数据采集可以通过[micadvisor_open](https://github.com/open-falcon/micadvisor_open)来做。

## 工作原理

micadvisor-open是基于open-falcon的docker容器资源监控插件，监控容器的cpu、内存、diskio以及网络io等，数据采集后上报到open-falcon

## 采集的指标

| Counters | Notes|
|-----|------|
|cpu.busy|cpu使用情况百分比|
|cpu.user|用户态使用的CPU百分比|
|cpu.system|内核态使用的CPU百分比|
|cpu.core.busy|每个cpu的使用情况|
|mem.memused.percent|内存使用百分比|
|mem.memused|内存使用原值|
|mem.memtotal|内存总量|
|mem.memused.hot|内存热使用情况|
|disk.io.read_bytes|磁盘io读字节数|
|disk.io.write_bytes|磁盘io写字节数|
|net.if.in.bytes|网络io流入字节数|
|net.if.in.packets|网络io流入包数|
|net.if.in.errors|网络io流入出错数|
|net.if.in.dropped|网络io流入丢弃数|
|net.if.out.bytes|网络io流出字节数|
|net.if.out.packets|网络io流出包数|
|net.if.out.errors|网络io流出出错数|
|net.if.out.dropped|网络io流出丢弃数|

## Contributors
- mengzhuo: QQ:296142139; MAIL:mengzhuo@xiaomi.com 

## 补充
- 另外一个docker metric采集的lib库：https://github.com/projecteru/eru-metric

