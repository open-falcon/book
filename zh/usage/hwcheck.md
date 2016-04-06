# 硬件监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

硬件的数据采集可以通过 [HWCheck](https://github.com/51web/hwcheck) 来做。

# HWCheck

rvadmin工具等组件实现硬件监控，需要安装falcon-agent

仅支持dell物理机，可以监控的指标有：

cpu 内存 阵列卡 物理磁盘 虚拟磁盘 阵列卡电池 BIOS 主板电池 风扇 电压 主板温度 cpu温度

# 如何安装

1. 配置dell官方repo，安装srvadmin等依赖包

```
#参考: http://linux.dell.com/repo/hardware/latest/
wget -q -O - http://linux.dell.com/repo/hardware/latest/bootstrap.cgi | bash

yum install srvadmin-omacore srvadmin-omcommon srvadmin-storage-cli smbios-utils-bin lm_sensors dmidecode cronie
# 启动srvadmin服务
/opt/dell/srvadmin/sbin/srvadmin-services.sh enable
/opt/dell/srvadmin/sbin/srvadmin-services.sh restart
# 配置lm-sensors
echo yes | /usr/sbin/sensors-detect
```

## 你也可以打包rpm来简化部署

```
git clone https://github.com/51web/hwcheck hwcheck-0.2
tar czf hwcheck-0.2.tar.gz hwcheck-0.2
rpmbuild -tb hwcheck-0.2.tar.gz
```


# 如何使用

## 参数说明

直接执行hwcheck不带参数默认会打印出详细的监控数据

```
hwcheck -d      # 打印metrics信息，即是push到falcon-agent的数据
        -p      # push数据到falcon-agent
        -s      # 设置push数据中的STEP数值，表示监控频率,默认值是600秒
        -m      # 指定单个metric
```

## 配置crontab

配置cron来定期检测，如：

```
cat /etc/cron.d/hwcheck
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/opt/dell/srvadmin/sbin:/opt/dell/srvadmin/bin
SHELL=/bin/bash

18 * * * * root /usr/bin/hwcheck -s 3600 -p >/dev/null 2>&1 &
```

表示每个小时执行一次检测，相应的STEP值被设置为3600


## falcon-portal中配置报警策略

hwcheck push到falcon-agent的metric均以 hw 打头，如hw.cpu_temp，除温度是实际的数值外，

其他metric的value中 0表示故障，1表示警告，2表示OK，例如在portal中配置如下策略：

| metric/tags/note             | condition |   max |  P  |
------------------------------ | --------- | ----- | --- |
| hw.bios [BIOS中C1E/Cstate未禁用] | all(#2)<2 | 1 | 4 |
| hw.board_temp [主板温度过高] | all(#3)>=35 | 1 | 4 |
| hw.cmos_bat [主板电池有问题] | all(#3)<2 |1 | 4 |
| hw.cpu [CPU可能故障]         | all(#2)==1 | 1 | 4 |
| hw.cpu [严重: CPU严重故障]   | all(#2)==0 | 2 | 0    |
| hw.fan [风扇出现故障]        | all(#3)<2 | 1 | 4     |
| hw.memory [内存可能故障]     | all(#1)==1 | 1 | 4    |
| hw.memory [严重: 内存严重故障] | all(#1)==0 | 2 | 0  |
| hw.pdisk [严重: 物理盘严重故障] | all(#1)==0 | 2 | 0     |
| hw.raidcard [阵列卡出现警告] | all(#2)==1 | 1 | 4    |
| hw.raidcard [严重: 阵列卡严重故障] | all(#1)==0 | 2 | 0  |
| hw.raidcard_bat [阵列卡电池出现警告] | all(#2)==1 | 1 | 4    |
| hw.raidcard_bat [严重: 阵列卡电池严重故障] | all(#2)==0 | 2 | 0  |
| hw.vdisk [磁盘阵列出现警告]    | all(#2)==1 | 1 | 4  |
| hw.vdisk [严重: 磁盘阵列严重故障] | all(#2)==0 | 2 | 0   |


