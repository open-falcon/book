<!-- toc -->


# 目录监控和进程详情监控实践

目录大小和进程详情的数据采集可用脚本[falcon-scripts](https://github.com/ZoneTong/falcon-scripts)来做。

收集的指标如下：

| 指标名 | 注释 |
|--------|------|
|du.bytes.used|目录大小，单位byte|
|proc.cpu|进程所占cpu，百分比|
|proc.mem|进程所占内存，单位byte|
|proc.io.in|进程io输入，单位byte|
|proc.io.out|进程io输出，单位byte|

## 工作原理

du.sh脚本借助du命令采集数据

proc.sh脚本分析/proc/$PID/status /proc/$PID/io等数据
