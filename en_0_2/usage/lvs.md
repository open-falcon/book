<!-- toc -->

# Lvs-metrics Introduction
Lvs-metrics is a LVS monitor module based on Open-falcon. Along with the agent and transfer of open-falcon, it can collect the state data of LVS service and push the collected data to the service of Open-falcon.

## Main Feature

Collecting the monitor data of LVS service through open source ipvs/netlink database from google and the files in proc, including the connection number of all the VIPs (active/inactive), the connection number of all the LVS machines (active/inactive) and the data size (packets) in the data traffic.

It does not hack the code of programm and cost little resources in the system.


## Collected Metrics

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


# Help
Please visit [lvs-metrics](https://github.com/mesos-utility/lvs-metrics) for more detailed instruction.

