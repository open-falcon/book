##Dockercontainer monitoring practice

We have introduced the usual monitoring data source in section of Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The data collection of docker container can be done by micadvisor_open.

##Operating principle

Micadvisor-open is the docker container resources monitoring plug-in based on open-falcon, which monitors the CPU, memory, diskio and net io etc. and collects the data and reports to open-falcon.

##The index collected

| Counters | Notes |
| -- | -- |
| cpu.busy | Cpu usage percent |
| cpu.user | Cpu usage percent in user mode |
| cpu.system | Cpu usage percent in kernel mode |
| cpu.core.busy | Every cpu usage percent |
| mem.memused.percent | Memory usage percent |
| mem.memused | Memory usage original value |
| mem.memtotal | Total memory |
| mem.memused.hot | Memory heat usage percent |
| disk.io.read_bytes | Disk io read bytes  |
| disk.io.write_bytes | Disk io write bytes |
| net.if.in.bytes | Net io in bytes  |
| net.if.in.packets | Net io in packets |
| net.if.in.errors | Net io in errors |
| net.if.in.dropped | Net io in dropped |
| net.if.out.bytes | Net io out bytes |
| net.if.out.packets | Net io out packets |
| net.if.out.errors | Net io out errors |
| net.if.out.dropped | Net io out dropped |

##Contributors
mengzhuo: QQ:296142139; MAIL:mengzhuo@xiaomi.com

##Supplement
* another lib bank of docker metric collection：https://github.com/projecteru/eru-metric

