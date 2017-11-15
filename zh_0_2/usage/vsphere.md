<!-- toc -->

## vsphere 监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

vsphere 的运行指标监控，可以通过脚本采集 vsphere 的各项状态，包括 Esxi，datastore，vm 等然后推送给 Open-Falcon 即可。

可以直接使用 [vsphere-monitor](https://github.com/freedomkk-qfeng/vsphere-monitor) 来实现对 vsphere 集群的监控指标采集。

#### 工作原理

vsphere-monitor 通过 [pyvmomi](https://github.com/vmware/pyvmomi) 采集 vsphere 集群的数据。只需要连接 vcenter 就可以采集集群内的包括 ESXi，datastore，vm 等各种监控数据。数据通过 open-falcon 的数据接口上报给 open-falcon。

#### 版本需求
python 2.7

#### 上报数据字段

 metric |  tag | type | note 
-----|------|------|------
datastore.capacity|datacetner=datacenter,datastore=datastore,type=type|GAUGE| 存储容量
datastore.free|datacetner=datacenter,datastore=datastore,type=type|GAUGE| 存储剩余容量
datastore.freePercent|datacetner=datacenter,datastore=datastore,type=type|GAUGE| 存储剩余容量
esxi.alive|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi 存活，值为 1，可以用来做 Nodata
esxi.net.if.in|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi 网络进流量（所有网卡总和）
esxi.net.if.out|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi 网络出流量（所有网卡总和）
esxi.memory.freePercent|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi 剩余内存百分比
esxi.memory.usage|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi 内存使用量
esxi.memory.capacity|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi 内存总量
esxi.cpu.usage|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi CPU 使用率
esxi.uptime|datacetner=datacenter，cluster_name=cluster_name，host=host|GAUGE|esxi uptime
vm.power|vm=vm_name|GAUGE|虚机是否开机，开机 = 1，关机 = 0，可以用来做 nodata
vm.net.if.in|vm=vm_name|GAUGE|虚机网络进流量（所有网卡总和）
vm.net.if.out|vm=vm_name|GAUGE|虚机网络出流量（所有网卡总和）
vm.datastore.io.write_latency|vm=vm_name|GAUGE|虚机存储 io 写延迟
vm.datastore.io.read_latency|vm=vm_name|GAUGE|虚机存储 io 读延迟
vm.datastore.io.write_numbers|vm=vm_name|GAUGE|虚机存储写 IOPS
vm.datastore.io.read_numbers|vm=vm_name|GAUGE|虚机存储读 IOPS
vm.datastore.io.write_bytes|vm=vm_name|GAUGE|虚机存储写流量
vm.datastore.io.read_bytes|vm=vm_name|GAUGE|虚机存储读流量
vm.memory.freePercent|vm=vm_name|GAUGE|虚机内存剩余百分比
vm.memory.usage|vm=vm_name|GAUGE|虚机内存量使用量
vm.memory.capacity|vm=vm_name|GAUGE|虚机内存量总量
vm.cpu.usage|vm=vm_name|GAUGE|虚机 cpu 使用量
vm.uptime|vm=vm_name|GAUGE|虚机 uptime


#### 安装
获取代码
```
git clone https://github.com/freedomkk-qfeng/vsphere-monitor.git
```
安装依赖
```
yum install -y python-virtualenv
cd vsphere-monitor
virtualenv ./env
./env/bin/pip install -r requirement.txt
```
#### 配置
修改配置文件 `config.py`
```
# falcon
endpoint = "vcenter" # 上报给 open-falcon 的 endpoint
push_api = "http://127.0.0.1:6060/api/push" # 上报的 http api 接口
interval = 60 # 上报的 step 间隔

# vcenter
host = "vcenter.host" # vcenter 的地址
user = "administrator@vsphere.local" # vcenter 的用户名
pwd = "password" # vcenter 的密码
port = 443 # vcenter 的端口

# esxi
esxi_names = [] # 需要采集的 esxi ，留空则全部采集

# datastore
datastore_names = [] # 需要采集的 datastore ，留空则全部采集 

# vm
vm_enable = True # 是否要采集虚拟机信息
vm_names = [     # 需要采集的虚拟机，留空则全部采集
            "vm1",
            "vm2",
            "vm3"
           ]
```

#### 运行
先尝试跑一下，假定 `vsphere-monitor` 放在 `/opt` 下
```
/opt/vsphere-monitor/env/bin/python /opt/vsphere-monitor/vsphere-monitor.py
```
没有问题的话，将他放入定时任务
```
crontab -e
0-59/1 * * * * /opt/vsphere-monitor/env/bin/python /opt/vsphere-monitor/vsphere-monitor.py
```

#### 问题
部分 vm 可能会采集不到 `vm.net.if.in` 和 `vm.net.if.out` 指标，这是因为网络指标是通过 `vsphere` 的 `PerformanceManager` 采集的。可以在 `vcenter` 中查看虚拟机的 `性能` 图表时，也会发现显示错误 `未指定衡量指标` 或者 `No Metric Specified` 。这是 `vmware` 的 bug，在 `vSphere 6.0 Update 1` 以上版本被修复

详见官方 Knowledge —— [在 VMware vSphere Client 6.0 中查看虚拟机网络的实时性能图表时显示错误：未指定衡量指标 (2125021)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2125021)

#### 截图
esxi 
![](https://github.com/freedomkk-qfeng/vsphere-monitor/blob/master/screenshots/esxi.png?raw=true)
vm
![](https://github.com/freedomkk-qfeng/vsphere-monitor/blob/master/screenshots/vm.png?raw=true)
