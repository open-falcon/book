# Esxi监控

VMware的主体机器(host machine)是运行Esxi作业系统。没有办法安装Open-Falcon agent来监控，所以不能用普通的方式来做监控。

Esxi作业系统设备的运行指标的采集，可以透过写脚本，通过SNMP协议来采集交换机的各项运行指标，包括内存占用、CPU使用、流量、磁盘用量等。[esxicollector](https://github.com/humorless/esxicollector)就是這樣子的腳本。

## 工作原理

esxicollector是一系列整理过的脚本。由[humorless](https://github.com/humorless/)设计开发。

esxicollector需要透过cronjob来配置。在一台可以跑cronjob的机器上，配置好cronjob。并且在esxi_collector.sh这个脚本中，写清楚要监控的设备、用来接受监控结果的Open-Falcon agent的位址。esxicollector就会照cronjob的时间间隔，预设是每分钟一次，定期地去采集Esxi作业系统设备的监控项，并透过上报到Open-Falcon的agent。

采集的metric列表：

* CPU利用率
  ```esxi.cpu.core```
* 内存總量/利用率
  ```esxi.cpu.memory.kliobytes.size```
  ```esxi.cpu.memory.kliobytes.used```
  ```esxi.cpu.memory.kliobytes.avail```
* 运行的进程数
  ```esxi.current.process```
* 登入的使用者数
  ```esxi.current.user```
* 虚拟机器数
  ```esxi.current.vhost```
* 磁盤總量/利用率
  ```esxi.df.size.kilobytes```
  ```esxi.df.used.percentage```
* 磁盤錯誤
  ```esxi.disk.allocationfailure```
* 網卡的輸出入流量/封包數
  ```esxi.net.in.octets```
  ```esxi.net.in.ucast.pkts```
  ```esxi.net.in.multicast.pkts```
  ```esxi.net.in.broadcast.pkts```
  ```esxi.net.out.octets```
  ```esxi.net.out.ucast.pkts```
  ```esxi.net.out.multicast.pkts```
  ```esxi.net.out.broadcast.pkts```
	

## 安装

从[这里](https://github.com/humorless/esxicollector)下载。

  1. 安装SNMP指令  
     ```yum -y install net-snmp net-snmp-utils```
  2. 下载VMware ESXi MIB档案，并且复制它们到资料夹```/usr/share/snmp/mibs```
  3. 设置SNMP的环境  
     ```mkdir ~/.snmp```  
     ```echo "mibs +ALL" > ~/.snmp/snmp.conf ```
  4. 在`esxi_collector.sh`填入合适的参数
  5. 设置cronjobs 
     ``` * * * * * esxi_collector.sh ```


## 延伸开发新的监控项

脚本 ```snmp_queries.sh``` 会呼叫基本的snmp指令，并且输出snmp执行完的结果。可以透过比较执行 ```60_esxi_*.sh```的结果，来设计新的脚本。
