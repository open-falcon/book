# 交换机监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

交换机设备的运行指标的采集，可以写脚本，通过SNMP协议来采集交换机的各项运行指标，包括内存占用、CPU使用、流量、ping延时等。

可以直接使用 [swcollector](https://github.com/gaochao1/swcollector) 来实现对交换机设备的监控指标采集。

## 工作原理

swcollector是一个Golang开发的后台程序，由@gaochao设计开发。

swcollector根据配置文件中，配置好的交换机IP列表或者IP网段，每隔一个固定的周期，通过SNMP协议，来采集交换机的运行指标，并上报给Open-Falcon。

采集的指标包括：

  * CPU利用率
  * 内存利用率
  * Ping延时
  * IfHCInOctets
  * IfHCOutOctets
  * IfHCInUcastPkts
  * IfHCOutUcastPkts

CPU和内存的OID私有，根据设备厂家和OS版本可能不同。目前测试过的设备：

  * Cisco IOS Software, Version 12.2(44)SE6
  * Cisco NX-OS, Version 6.2(10)
  * Huawei Versatile Routing Platform Software VRP (R) software, Version 8.60
  * H3C Comware Platform Software, Software Version 5.20
  * H3C Comware Platform Software, Software Version 7.1.045

## 部署说明

swcollector需要部署到有交换机SNMP访问权限的服务器上。

使用Go原生的ICMP协议进行Ping探测，swcollector需要root权限运行。

Huawei交换机使用原生SNMP协议各种报错，对Huawei交换机暂时解决方案使用系统snmpwalk命令采集数据。


