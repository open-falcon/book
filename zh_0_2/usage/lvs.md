# lvs-metrics 简介
lvs-metrics是一个基于open-falcon的LVS监控插件，通过这个插件，结合open-falcon agent/transfer，可以采集LVS服务状态，并将采集信息自动上报给open-falcon服务端

## 主要功能

通过google开源的ipvs/netlink库及proc下文件采集lvs的监控信息，包括所有VIP的连接数(活跃/非活跃)/LVS主机的连接数(活跃/非活跃).进出数据包数/字节数.

对应用程序代码无侵入，几乎不占用系统资源。


## 采集指标

| Counters | Type | Notes |
|-----|-----|-----|
| lvs.in.bytes | GUAGE | network in bytes per host |
| lvs.out.bytes | GUAGE | network out bytes per host |
| lvs.in.packets | GUAGE | network in packets per host |
| lvs.out.packets | GUAGE | network out packets per host |
| lvs.total.conns | GUAGE | lvs total connections per vip now |
| lvs.active.conn | GUAGE | lvs active connections per vip now |
| lvs.inact.conn | GUAGE | lvs inactive connections per vip now |
| lvs.realserver.num | GUAGE | lvs live realserver num per vip now |
| lvs.vip.conns | COUNTER | lvs conns counter from service start per vip |
| lvs.vip.inbytes | COUNTER | lvs inbytes counter from service start per vip |
| lvs.vip.outbytes | COUNTER | lvs outpkts counter from service start per vip |
| lvs.vip.inpkts | COUNTER | lvs inpkts counter from service start per vip |
| lvs.vip.outpkts | COUNTER | lvs outpkts counter from service start per vip |


# 使用帮助
详细的使用方法常见：[lvs-metrics](https://github.com/mesos-utility/lvs-metrics)

