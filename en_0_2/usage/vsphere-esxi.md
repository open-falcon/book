### falcon-vsphere
This is an open-falcon agent for monitoring Vsphere and all esxi performance metrics overseen by Vsphere。Address:[https://github.com/tpinellia/vsphere](https://github.com/tpinellia/vsphere)
#### 1.Features
- Support multi-vsphere collection
- Support merging/splitting of vsphere and esxi monitoring items,custom endpoint or monitor headers
- The esxi monitor includes both base and extended parts, which can be optionally enabled and configured
- Configuration supports hot loading (for example, adding vsphere, adding/deleting extension monitoring, etc.) without restarting the agent
#### 2.Instructions
- export WorkDir="$HOME/falcon-vsphere"
- mkdir -p $WorkDir
- tar -xzvf falcon-vsphere-x.x.x.tar.gz -C $WorkDir
- cd $WorkDir
- ./control start
#### 3.Configuration
```
{
    "debug": false,
    "extend": "./extend.json",              #extend items     
    "heartbeat": { 
        "enabled": true,                    #enabled here must be set to "true"
        "addr": "127.0.0.1:6030",           #address of hbs and the port belongs to rpc of hbs
        "timeout": 1000                     #time-out setting for hbs connection, measured in millisecond
    },
    "transfer": {
        "enabled": true,                   
        "addrs": [              
            "127.0.0.1:8433"                #the address of Transfer; and the port belongs to rpc of Transfer; more than one Transfer addresses are supported here because Agent will guarantee HA
        ],
        "interval": 60,                     #collecting interval measured in second
        "timeout": 1000                     #time-out setting for Transfer connection, measured in millisecond          
    },
    "vsphere": [
        {
            "hostname": "VC-1.1.1.1",       #Agent collects data and send them to Transfer and then "endpoint" becomes the hostname from "hostname"
            "ip": "1.1.1.1",                #Vsphere IP
            "addr": "https://1.1.1.1/sdk",  #Vsphere SDK Address
            "user":"vsphere1-user",         #Vsphere username
            "pwd":"vsphere1-pwd",           #Vsphere password
            "port": 443,                    #Vsphere port
            "split": true,                  #Split or not。
            "endpointhead": "ESXI-",        #When split is true, it will be used as an extension header to the esxi endpoint
            "metrichead":"esxi.",           #When split is false, it will be used as an extension header to the esxi's items
            "extend": true                  #use extended items or not
        },
        {
            "hostname": "VC-2.2.2.2",
            "ip": "2.2.2.2",
            "addr": "https://2.2.2.2/sdk",
            "user":"vsphere2-user",
            "pwd":"vsphere2-pwd",
            "port": 443,
            "split": true,
            "endpointhead": "ESXI-",
            "metrichead":"esxi.",
            "extend": true
        }
    ]
}
```
#### 4.Items
##### 4.1. Base

|COUNTER|INSTRUCTIONS|
|---|---|
|agent.alive|1|
|agent.power|1:poweredOff;2:poweredOn;3:standBy;4:unknown|
|agent.status|1:The status is unknown.;2:The entity is OK. ;3:The entity definitely has a problem. ;4:The entity might have a problem. |
|agent.uptime|The system uptime of the host in seconds.|
|cpu.busy|Percentage of time that the CPU or CPUs were busy|
|cpu.free.average|Aggregated CPU free across all cores on the host in Hz|
|cpu.total|Aggregated CPU across all cores on the host in Hz|
|cpu.usage.average|Aggregated CPU usage across all cores on the host in Hz. This is only available if the host is connected. |
|datastore.totalReadLatency.average|Average amount of time for a read operation from the datastore. Total latency = kernel latency + device latency.|
|datastore.totalWriteLatency.average|Average amount of time for a write operation to the datastore. Total latency = kernel latency + device latency.|
|df.bytes.free.percent|Percent of free space(Per Datastores)|
|df.bytes.free|Available space of this datastore, in bytes.(Per Datastores)|
|df.bytes.total|Total space of this datastore, in bytes.(Per Datastores)|
|df.bytes.used.Percent|Percent of used space(Per Datastores)|
|df.bytes.used|Used space of this datastore, in bytes.(Per Datastores)|
|df.statistics.free|Available space of this datastore, in bytes.(All Datastores)|
|df.statistics.free.percent|Percent of free space(All Datastores)|
|df.statistics.total|Total space of all datastore(All Datastores)|
|df.statistics.used|Used space of this datastore, in bytes.(All Datastores)|
|df.statistics.used.percent|Percent of used space(All Datastores)|
|mem.memfree|Free mem on the host|
|mem.memfree.percent|Percent of free mem|
|mem.memtotal|Total memory on the host|
|mem.memused|Physical memory usage on the host in MB.|
|mem.memused.percent|Percent of used mem|
|net.bytesRx.average|Average amount of data received per second.|
|net.bytesTx.average|Average amount of data transmitted per second.|

##### 4.2. extended

**Type Instructions**

|NAME|DESCRIPTION|
|-|-|
|absolute|Represents an actual value, level, or state of the counter. For example, the “uptime” counter (system group) represents the actual number of seconds since startup. The “capacity” counter represents the actual configured size of the specified datastore. In other words, number of samples, samplingPeriod, and intervals have no bearing on an “absolute” counter“s value.|
|delta|Represents an amount of change for the counter during the samplingPeriod as compared to the previous interval. The first sampling interval|
|rate|Represents a value that has been normalized over the samplingPeriod, enabling values for the same counter type to be compared, regardless of interval. For example, the number of reads per second.|

- hbr

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|hbr.hbrNetTx.average|Average amount of data transmitted per second|rate|
|hbr.hbrNetRx.average|Kilobytes per second of outgoing host-based replication network traffic (for this virtual machine or host)|rate| 
|hbr.hbrNumVms.average|Number of powered-on virtual machines running on this host that currently have host-based replication protection enabled.|absolute|

- rescpu

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|rescpu.runav15.latest|CPU running average over 15 minutes|absolute|
|rescpu.actav1.latest|CPU running average over 1 minute|absolute|
|rescpu.actpk15.latest|CPU active peak over 15 minutes|absolute|
|rescpu.actav5.latest|CPU running average over 5 minutes|absolute|
|rescpu.runpk1.latest|CPU running peak over 1 minute|absolute|
|rescpu.maxLimited5.latest|Amount of CPU resources over the limit that were refused, average over 5 minutes|absolute|
|rescpu.actpk1.latest|CPU active peak over 1 minute|absolute|
|rescpu.sampleCount.latest|Group CPU sample count.|absolute|
|rescpu.samplePeriod.latest|Group CPU sample period.|absolute|
|rescpu.maxLimited1.latest|Amount of CPU resources over the limit that were refused, average over 1 minute|absolute|
|rescpu.runpk15.latest|CPU running peak over 15 minutes|absolute|
|rescpu.maxLimited15.latest|Amount of CPU resources over the limit that were refused, average over 15 minutes|absolute|
|rescpu.runav5.latest|CPU running average over 5 minutes|absolute|
|rescpu.runav1.latest|CPU running average over 1 minute|absolute|
|rescpu.actav15.latest|CPU active average over 15 minutes|absolute|
|rescpu.actpk5.latest|CPU active peak over 5 minutes|absolute|
|rescpu.runpk5.latest|CPU running peak over 5 minutes|absolute|

- storagePath

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|storagePath.totalReadLatency.average|Average amount of time for a read issued on the storage path. Total latency = kernel latency + device latency.|absolute|
|storagePath.commandsAveraged.average|Average number of commands issued per second on the storage path during the collection interval|rate|
|storagePath.numberReadAveraged.average|Average number of read commands issued per second on the storage path during the collection interval|rate|
|storagePath.write.average|Rate of writing data on the storage path|rate|
|storagePath.maxTotalLatency.latest|Highest latency value across all storage paths used by the host|absolute|
|storagePath.read.average|Rate of reading data on the storage path|rate|
|storagePath.numberWriteAveraged.average|Average number of write commands issued per second on the storage path during the collection interval|rate|
|storagePath.totalWriteLatency.average|Average amount of time for a write issued on the storage path. Total latency = kernel latency + device latency.|absolute|
- storageAdapter

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|storageAdapter.read.average|Rate of reading data by the storage adapter|rate|
|storageAdapter.numberReadAveraged.average|Average number of read commands issued per second by the storage adapter during the collection interval|rate| |storageAdapter.maxTotalLatency.latest|Highest latency value across all storage adapters used by the host|rate|
|storageAdapter.numberWriteAveraged.average|Average number of write commands issued per second by the storage adapter during the collection interval|rate|
|storageAdapter.totalWriteLatency.average|Average amount of time for a write operation by the storage adapter. Total latency = kernel latency + device latency.|absolute|
|storageAdapter.totalReadLatency.average|Average amount of time for a read operation by the storage adapter. Total latency = kernel latency + device latency.|absolute|
|storageAdapter.commandsAveraged.average|Average number of commands issued per second by the storage adapter during the collection interval|rate|
|storageAdapter.write.average|Rate of writing data by the storage adapter|rate|
- power

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|power.power.average|Current power usage.|rate|
|power.powerCap.average|Maximum allowed power usage.|rate|
|power.energy.summation|Total energy used since last stats reset.|rate|
- sys

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|sys.resourceCpuUsage.average|Amount of CPU used by the Service Console and other applications during the interval by the Service Console and other applications.|rate|
|sys.resourceCpuUsage.maximum|Amount of CPU used by the Service Console and other applications during the interval by the Service Console and other applications.|rate|
|sys.resourceMemMapped.latest|Memory mapped by the system resource group|absolute|
|sys.resourceMemAllocShares.latest|Memory allocation shares of the system resource group|absolute|
|sys.resourceMemCow.latest|Memory shared by the system resource group|absolute|
|sys.resourceMemShared.latest|Memory saved due to sharing by the system resource group|absolute|
|sys.resourceCpuAct1.latest|CPU active average over 1 minute of the system resource group|absolute|
|sys.resourceCpuUsage.minimum|Amount of CPU used by the Service Console and other applications during the interval by the Service Console and other applications.|rate|
|sys.resourceMemZero.latest|Zero filled memory used by the system resource group|absolute|
|sys.resourceCpuAct5.latest|CPU active average over 5 minutes of the system resource group|absolute|
|sys.resourceCpuMaxLimited1.latest|CPU maximum limited over 1 minute of the system resource group|absolute|
|sys.resourceFdUsage.latest|Number of file descriptors used by the system resource group|absolute|
|sys.resourceMemConsumed.latest|Memory consumed by the system resource group  |absolute|
|sys.resourceCpuRun5.latest|CPU running average over 5 minutes of the system resource group|absolute|
|sys.uptime.latest|Total time elapsed, in seconds, since last system startup  |absolute|
|sys.resourceCpuAllocMin.latest|CPU allocation reservation (in MHz) of the system resource group|absolute|
|sys.resourceCpuAllocShares.latest|CPU allocation shares of the system resource group|absolute|
|sys.resourceCpuMaxLimited5.latest|CPU maximum limited over 5 minutes of the system resource group|absolute|
|sys.resourceMemOverhead.latest|Overhead memory consumed by the system resource group|absolute|
|sys.resourceCpuUsage.none|Amount of CPU used by the Service Console and other applications during the interval by the Service Console and other applications.|rate|
|sys.resourceCpuRun1.latest|CPU running average over 1 minute of the system resource group|absolute|
|sys.resourceMemSwapped.latest|Memory swapped out by the system resource group|absolute|
|sys.resourceMemTouched.latest|Memory touched by the system resource group|absolute|
|sys.resourceMemAllocMax.latest|Memory allocation limit (in KB) of the system resource group|absolute|
|sys.resourceMemAllocMin.latest|Memory allocation reservation (in KB) of the system resource group|absolute|
- net

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|net.unknownProtos.summation|Number of frames with unknown protocol received during the sampling interval|delta|
|net.packetsRx.summation|Total number of packets received on all virtual machines running on the host.|delta|
|net.received.average|The rate at which data is received across each physical NIC instance on the host.|rate|
|net.errorsRx.summation|Number of packets with errors received during the sampling interval.|delta|
|net.transmitted.average|The rate at which data is transmitted across each physical NIC instance on the host.|rate|
|net.usage.none|Sum of data transmitted and received across all physical NIC instances connected to the host.|rate|
|net.bytesRx.average|Average amount of data received per second.|rate|
|net.multicastTx.summation|Number of multicast packets transmitted during the sampling interval.|delta|
|net.droppedTx.summation|Number of transmitted packets dropped during the collection interval.|delta|
|net.usage.minimum|Sum of data transmitted and received across all physical NIC instances connected to the host.|rate|
|net.multicastRx.summation|Number of multicast packets received during the sampling interval.|delta|
|net.bytesTx.average|Average amount of data transmitted per second.|rate|
|net.broadcastRx.summation|Number of broadcast packets received during the sampling interval.|delta|
|net.packetsTx.summation|Number of packets transmitted across each physical NIC instance on the host.|delta|
|net.errorsTx.summation|Number of packets with errors transmitted during the sampling interval.|delta|
|net.broadcastTx.summation|Number of broadcast packets transmitted during the sampling interval.|delta|
|net.usage.maximum|Sum of data transmitted and received across all physical NIC instances connected to the host.|rate|
|net.usage.average|Sum of data transmitted and received across all physical NIC instances connected to the host.|rate|
|net.droppedRx.summation|Number of received packets dropped during the collection interval.|delta|
- disk

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|disk.usage.maximum|Aggregated disk I/O rate. For hosts, this metric includes the rates for all virtual machines running on the host during the collection interval.|rate|
|disk.deviceWriteLatency.average|Average amount of time, in milliseconds, to write to the physical device|absolute|
|disk.queueWriteLatency.average|Average amount of time spent in the VMkernel queue, per SCSI write command, during the collection interval|absolute|
|disk.commands.summation|Number of SCSI commands issued during the collection interval|delta|
|disk.busResets.summation|Number of SCSI-bus reset commands issued during the collection interval|delta|
|disk.numberRead.summation|Number of times data was read from each LUN on the host.|delta|
|disk.deviceLatency.average|Average amount of time, in milliseconds, to complete a SCSI command from the physical device|absolute|
|disk.deviceReadLatency.average|Average amount of time, in milliseconds, to read from the physical device|absolute|
|disk.usage.average|Aggregated disk I/O rate. For hosts, this metric includes the rates for all virtual machines running on the host during the collection interval.|rate|
|disk.numberWriteAveraged.average|Average number of write commands issued per second to the datastore during the collection interval.|rate|
|disk.usage.none|Aggregated disk I/O rate. For hosts, this metric includes the rates for all virtual machines running on the host during the collection interval.|rate|
|disk.totalWriteLatency.average|Average amount of time taken during the collection interval to process a SCSI write command issued by the guest OS to the virtual machine|absolute|
|disk.queueLatency.average|Average amount of time spent in the VMkernel queue, per SCSI command, during the collection interval.|absolute|
|disk.kernelLatency.average|Average amount of time, in milliseconds, spent by VMkernel to process each SCSI command|absolute|
|disk.write.average|Rate at which data is written to each LUN on the host.write rate = # blocksRead per second x blockSize|rate|
|disk.numberWrite.summation|Number of times data was written to each LUN on the host.|delta|
|disk.commandsAveraged.average|Average number of SCSI commands issued per second during the collection interval|rate|
|disk.totalLatency.average|Average amount of time taken during the collection interval to process a SCSI command issued by the guest OS to the virtual machine.|absolute|
|disk.totalReadLatency.average|Average amount of time taken during the collection interval to process a SCSI read command issued from the guest OS to the virtual machine|absolute|
|disk.kernelWriteLatency.average|Average amount of time, in milliseconds, spent by VMkernel to process each SCSI write command|absolute|
|disk.maxTotalLatency.latest|Highest latency value across all disks used by the host. Latency measures the time taken to process a SCSI command issued by the guest OS to the virtual machine. The kernel latency is the time VMkernel takes to process an IO request. The device latency is the time it takes the hardware to handle the request.|absolute|
|disk.numberReadAveraged.average|Number of times data was read from each LUN on the host.|delta|
|disk.read.average|Rate at which data is read from each LUN on the host.read rate = # blocksRead per second x blockSize|rate|
|disk.queueReadLatency.average|Average amount of time spent in the VMkernel queue, per SCSI read command, during the collection interval|absolute|
|disk.usage.minimum|Aggregated disk I/O rate. For hosts, this metric includes the rates for all virtual machines running on the host during the collection interval.|rate|
|disk.commandsAborted.summation|Number of SCSI commands aborted during the collection interval|delta|
|disk.maxQueueDepth.average|Maximum queue depth.|absolute|
|disk.kernelReadLatency.average|Average amount of time, in milliseconds, spent by VMkernel to process each SCSI read command|absolute|
- cpu

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|cpu.reservedCapacity.average|Total CPU capacity reserved by virtual machines |absolute|
|cpu.usagemhz.average|Sum of the actively used CPU of all powered on virtual machines on a host. The maximum possible value is the frequency of the processors multiplied by the number of processors. For example, if you have a host with four 2GHz CPUs running a virtual machine that is using 4000MHz, the host is using two CPUs completely.4000 / (4 x 2000) = 0.50|rate|
|cpu.usage.none|Actively used CPU of the host, as a percentage of the total available CPU. Active CPU is approximately equal to the ratio of the used CPU to the available CPU. available CPU = # of physical CPUs x clock rate.100% represents all CPUs on the host. For example, if a four-CPU host is running a virtual machine with two CPUs, and the usage is 50%, the host is using two CPUs completely.|rate|
|cpu.coreUtilization.maximum|CPU utilization of the corresponding core (if hyper-threading is enabled) as a percentage during the interval (A core is utilized if either or both of its logical CPUs are utilized).|rate|
|cpu.costop.summation|Time the virtual machine is ready to run, but is unable to run due to co-scheduling constraints|delta|
|cpu.totalCapacity.average|Total CPU capacity reserved by and available for virtual machines|absolute|
|cpu.latency.average|Percent of time the virtual machine is unable to run because it is contending for access to the physical CPU(s)|rate|
|cpu.usage.average|Actively used CPU of the host, as a percentage of the total available CPU. Active CPU is approximately equal to the ratio of the used CPU to the available CPU. available CPU = # of physical CPUs x clock rate.100% represents all CPUs on the host. For example, if a four-CPU host is running a virtual machine with two CPUs, and the usage is 50%, the host is using two CPUs completely.|rate|
|cpu.utilization.maximum|CPU utilization as a percentage during the interval (CPU usage and CPU utilization might be different due to power management technologies or hyper-threading)|rate|
|cpu.coreUtilization.minimum|CPU utilization of the corresponding core (if hyper-threading is enabled) as a percentage during the interval (A core is utilized if either or both of its logical CPUs are utilized).|rate|
|cpu.wait.summation|Total CPU time spent in wait state.The wait total includes time spent the CPU Idle, CPU Swap Wait, and CPU I/O Wait states.|rate|
|cpu.swapwait.summation|CPU time spent waiting for swap-in.|delta|
|cpu.ready.summation|Time that the virtual machine was ready, but could not get scheduled to run on the physical CPU during last measurement interval. CPU ready time is dependent on the number of virtual machines on the host and their CPU loads.|delta|
|cpu.utilization.average|CPU utilization as a percentage during the interval (CPU usage and CPU utilization might be different due to power management technologies or hyper-threading)|rate|
|cpu.used.summation|Time accounted to the virtual machine. If a system service runs on behalf of this virtual machine, the time spent by that service (represented by cpu.system) should be charged to this virtual machine. If not, the time spent (represented by cpu.overlap) should not be charged against this virtual machine.|delta|
|cpu.utilization.none|CPU utilization as a percentage during the interval (CPU usage and CPU utilization might be different due to power management technologies or hyper-threading)|rate|
|cpu.idle.summation|Total time that the CPU spent in an idle state|delta|
|cpu.coreUtilization.none|CPU utilization of the corresponding core (if hyper-threading is enabled) as a percentage during the interval (A core is utilized if either or both of its logical CPUs are utilized).|rate|
|cpu.coreUtilization.average|CPU utilization of the corresponding core (if hyper-threading is enabled) as a percentage during the interval (A core is utilized if either or both of its logical CPUs are utilized).|rate|
|cpu.readiness.average|Percentage of time that the virtual machine was ready, but could not get scheduled to run on the physical CPU.|rate|
|cpu.usage.maximum|Actively used CPU of the host, as a percentage of the total available CPU. Active CPU is approximately equal to the ratio of the used CPU to the available CPU. available CPU = # of physical CPUs x clock rate.100% represents all CPUs on the host. For example, if a four-CPU host is running a virtual machine with two CPUs, and the usage is 50%, the host is using two CPUs completely.|rate|
|cpu.usagemhz.none|Host - Sum of the actively used CPU of all powered on virtual machines on a host. The maximum possible value is the frequency of the processors multiplied by the number of processors. For example, if you have a host with four 2GHz CPUs running a virtual machine that is using 4000MHz, the host is using two CPUs completely.4000 / (4 x 2000) = 0.50|rate|
|cpu.usage.minimum|Actively used CPU of the host, as a percentage of the total available CPU. Active CPU is approximately equal to the ratio of the used CPU to the available CPU. available CPU = # of physical CPUs x clock rate.100% represents all CPUs on the host. For example, if a four-CPU host is running a virtual machine with two CPUs, and the usage is 50%, the host is using two CPUs completely.|rate|
|cpu.demand.average|The amount of CPU resources a virtual machine would use if there were no CPU contention or CPU limit|absolute|
|cpu.usagemhz.maximum|Host - Sum of the actively used CPU of all powered on virtual machines on a host. The maximum possible value is the frequency of the processors multiplied by the number of processors. For example, if you have a host with four 2GHz CPUs running a virtual machine that is using 4000MHz, the host is using two CPUs completely.4000 / (4 x 2000) = 0.50|rate|
|cpu.utilization.minimum|CPU utilization as a percentage during the interval (CPU usage and CPU utilization might be different due to power management technologies or hyper-threading)|rate|
|cpu.usagemhz.minimum|Host - Sum of the actively used CPU of all powered on virtual machines on a host. The maximum possible value is the frequency of the processors multiplied by the number of processors. For example, if you have a host with four 2GHz CPUs running a virtual machine that is using 4000MHz, the host is using two CPUs completely.4000 / (4 x 2000) = 0.50|rate|
- mem

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|mem.llSwapOut.maximum|Amount of memory swapped-out to host cache|absolute|
|mem.swapin.maximum|Sum of swapin values for all powered-on virtual machines on the host.|bsolute|
|mem.compressed.average|Amount of memory reserved by userworlds.|absolute|
|mem.overhead.none|Total of all overhead metrics for powered-on virtual machines, plus the overhead of running vSphere services on the host.|absolute|
|mem.heap.maximum|VMkernel virtual address space dedicated to VMkernel main heap and related data|absolute|
|mem.unreserved.none|Amount of memory that is unreserved. Memory reservation not used by the Service Console, VMkernel, vSphere services and other powered on VMs’ user-specified memory reservations and overhead memory. This statistic is no longer relevant to virtual machine admission control, as reservations are now handled through resource pools.|absolute|
|mem.reservedCapacity.average|Total amount of memory reservation used by powered-on virtual machines and vSphere services on the host.|absolute|
|mem.swapoutRate.average|Rate at which memory is being swapped from active memory to disk during the current interval. This counter applies to virtual machines and is generally more useful than the swapout counter to determine if the virtual machine is running slow due to swapping, especially when looking at real-time statistics.|rate|
|mem.vmmemctl.none|The sum of all vmmemctl values for all powered-on virtual machines, plus vSphere services on the host. If the balloon target value is greater than the balloon value, the VMkernel inflates the balloon, causing more virtual machine memory to be reclaimed. If the balloon target value is less than the balloon value, the VMkernel deflates the balloon, which allows the virtual machine to consume additional memory if needed.|absolute|
|mem.vmfs.pbc.sizeMax.latest|Maximum size the VMFS Pointer Block Cache can grow to|absolute|
|mem.consumed.maximum|Amount of machine memory used on the host. Consumed memory includes Includes memory used by the Service Console, the VMkernel, vSphere services, plus the total consumed metrics for all running virtual machines. host consumed memory = total host memory - free host memory|absolute|
|mem.swapin.minimum|Sum of swapin values for all powered-on virtual machines on the host.|absolute|
|mem.vmmemctl.maximum|The sum of all vmmemctl values for all powered-on virtual machines, plus vSphere services on the host. If the balloon target value is greater than the balloon value, the VMkernel inflates the balloon, causing more virtual machine memory to be reclaimed. If the balloon target value is less than the balloon value, the VMkernel deflates the balloon, which allows the virtual machine to consume additional memory if needed.|absolute|
|mem.llSwapUsed.maximum|Space used for caching swapped pages in the host cache|absolute|
|mem.sharedcommon.average|Amount of machine memory that is shared by all powered-on virtual machines and vSphere services on the host.Subtract this metric from the shared metric to gauge how much machine memory is saved due to sharing: shared - sharedcommon = machine memory (host memory) savings (KB)|absolute|
|mem.active.minimum|Sum of all active metrics for all powered-on virtual machines plus vSphere services (such as COS, vpxa) on the host.|absolute|
|mem.sysUsage.none|Amount of host physical memory used by VMkernel for core functionality, such as device drivers and other internal uses. Does not include memory used by virtual machines or vSphere services.|absolute|
|mem.swapin.average|Sum of swapin values for all powered-on virtual machines on the host.|absolute|
|mem.zero.none|Sum of zero metrics for all powered-on virtual machines, plus vSphere services on the host.|absolute|
|mem.llSwapOut.average|Amount of memory swapped-out to host cache|absolute|
|mem.granted.minimum|Sum of all granted metrics for all powered-on virtual machines, plus machine memory for vSphere services on the host.|absolute|
|mem.granted.none|Sum of all granted metrics for all powered-on virtual machines, plus machine memory for vSphere services on the host.|absolute|
|mem.consumed.minimum|Amount of machine memory used on the host. Consumed memory includes Includes memory used by the Service Console, the VMkernel, vSphere services, plus the total consumed metrics for all running virtual machines.host consumed memory = total host memory - free host memory|absolute|
|mem.vmmemctl.minimum|The sum of all vmmemctl values for all powered-on virtual machines, plus vSphere services on the host. If the balloon target value is greater than the balloon value, the VMkernel inflates the balloon, causing more virtual machine memory to be reclaimed. If the balloon target value is less than the balloon value, the VMkernel deflates the balloon, which allows the virtual machine to consume additional memory if needed.|absolute|
|mem.consumed.average|Amount of machine memory used on the host. Consumed memory includes Includes memory used by the Service Console, the VMkernel, vSphere services, plus the total consumed metrics for all running virtual machines.host consumed memory = total host memory - free host memory|absolute|
|mem.active.none|Sum of all active metrics for all powered-on virtual machines plus vSphere services (such as COS, vpxa) on the host.|absolute|
|mem.vmfs.pbc.overhead.latest|Amount of VMFS heap used by the VMFS PB Cache|  absolute|
|mem.swapout.average|Sum of swapout metrics from all powered-on virtual machines on the host.|absolute|
|mem.usage.minimum|Percentage of available machine memory:consumed ÷ machine-memory-size|absolute|
|mem.consumed.none|Amount of machine memory used on the host. Consumed memory includes Includes memory used by the Service Console, the VMkernel, vSphere services, plus the total consumed metrics for all running virtual machines.host consumed memory = total host memory - free host memory|absolute|
|mem.heapfree.none|Free address space in the VMkernel main heap.Varies based on number of physical devices and configuration options. There is no direct way for the user to increase or decrease this statistic. For informational purposes only: not useful for performance monitoring.|absolute|
|mem.vmfs.pbc.size.latest|Space used for holding VMFS Pointer Blocks in memory|absolute|
|mem.llSwapOut.none|Amount of memory swapped-out to host cache|absolute|
|mem.totalCapacity.average|Total amount of memory reservation used by and available for powered-on virtual machines and vSphere services on the host|absolute|
|mem.sysUsage.minimum|Amount of host physical memory used by VMkernel for core functionality, such as device drivers and other internal uses. Does not include memory used by virtual machines or vSphere services.|absolute|
|mem.granted.average|Sum of all granted metrics for all powered-on virtual machines, plus machine memory for vSphere services on the host.|absolute|
|mem.activewrite.average|Estimate for the amount of memory actively being written to by the virtual machine.|absolute|
|mem.unreserved.average|Amount of memory that is unreserved. Memory reservation not used by the Service Console, VMkernel, vSphere services and other powered on VMs’ user-specified memory reservations and overhead memory. This statistic is no longer relevant to virtual machine admission control, as reservations are now handled through resource pools.|absolute|
|mem.llSwapIn.average|Amount of memory swapped-in from host cache|absolute|
|mem.overhead.maximum|Total of all overhead metrics for powered-on virtual machines, plus the overhead of running vSphere services on the host.|absolute|
|mem.llSwapOut.minimum|Amount of memory swapped-out to host cache|absolute|
|mem.shared.none|Sum of all shared metrics for all powered-on virtual machines, plus amount for vSphere services on the host. The host's shared memory may be larger than the amount of machine memory if memory is overcommitted (the aggregate virtual machine configured memory is much greater than machine memory). The value of this statistic reflects how effective transparent page sharing and memory overcommitment are for saving machine memory.|absolute|
|mem.usage.average|Percentage of available machine memory:consumed ÷ machine-memory-size|absolute|
|mem.llSwapInRate.average|Rate at which memory is being swapped from host cache into active memory|rate|
|mem.sysUsage.average|Amount of host physical memory used by VMkernel for core functionality, such as device drivers and other internal uses. Does not include memory used by virtual machines or vSphere services.|absolute|
|mem.lowfreethreshold.average|Threshold of free host physical memory below which ESX/ESXi will begin reclaiming memory from virtual machines through ballooning and swapping|absolute|
|mem.zero.average|Sum of zero metrics for all powered-on virtual machines, plus vSphere services on the host.|absolute|
|mem.vmfs.pbc.capMissRatio.latest|Trailing average of the ratio of capacity misses to compulsory misses for the VMFS PB Cache|absolute|
|mem.state.latest|One of four threshold levels representing the percentage of free memory on the host. The counter value determines swapping and ballooning behavior for memory reclamation.0 (high) Free memory >= 6% of machine memory minus Service Console memory;1 (soft) 4%;2 (hard) 2%; 3 (low) 1%; 0 (high) and 1 (soft): Ballooning is favored over swapping.2 (hard) and 3 (low):  Swapping is favored over ballooning.|absolute|
|mem.heap.average|VMkernel virtual address space dedicated to VMkernel main heap and related data|absolute|
|mem.heapfree.average|Free address space in the VMkernel main heap.Varies based on number of physical devices and configuration options. There is no direct way for the user to increase or decrease this statistic. For informational purposes only: not useful for performance monitoring.|absolute|
|mem.granted.maximum|Sum of all granted metrics for all powered-on virtual machines, plus machine memory for vSphere services on the host.|absolute|
|mem.llSwapIn.maximum|Amount of memory swapped-in from host cache|absolute|
|mem.llSwapUsed.none|Space used for caching swapped pages in the host cache|absolute|
|mem.swapout.minimum|Sum of swapout metrics from all powered-on virtual machines on the host.|absolute|
|mem.active.average|Sum of all active metrics for all powered-on virtual machines plus vSphere services (such as COS, vpxa) on the host.|absolute|
|mem.llSwapIn.minimum|Amount of memory swapped-in from host cache|absolute|
|mem.overhead.minimum|Total of all overhead metrics for powered-on virtual machines, plus the overhead of running vSphere services on the host.|absolute|
|mem.llSwapUsed.minimum|Space used for caching swapped pages in the host cache|absolute|
|mem.swapinRate.average|Rate at which memory is swapped from disk into active memory during the interval. This counter applies to virtual machines and is generally more useful than the swapin counter to determine if the virtual machine is running slow due to swapping, especially when looking at real-time statistics.|rate|
|mem.heapfree.maximum|Free address space in the VMkernel main heap.Varies based on number of physical devices and configuration options. There is no direct way for the user to increase or decrease this statistic. For informational purposes only: not useful for performance monitoring.|absolute|
|mem.shared.average|Sum of all shared metrics for all powered-on virtual machines, plus amount for vSphere services on the host. The host's shared memory may be larger than the amount of machine memory if memory is overcommitted (the aggregate virtual machine configured memory is much greater than machine memory). The value of this statistic reflects how effective transparent page sharing and memory overcommitment are for saving machine memory.|absolute|
|mem.zero.maximum|Sum of zero metrics for all powered-on virtual machines, plus vSphere services on the host.|absolute|
|mem.swapout.maximum|Sum of swapout metrics from all powered-on virtual machines on the host.|absolute|
|mem.swapin.none|Sum of swapin values for all powered-on virtual machines on the host.|absolute|
|mem.heap.minimum|VMkernel virtual address space dedicated to VMkernel main heap and related data|absolute|
|mem.decompressionRate.average|Rate of memory decompression for the virtual machine.|rate|
|mem.compressionRate.average|Rate of memory compression for the virtual machine.|rate|
|mem.shared.minimum|Sum of all shared metrics for all powered-on virtual machines, plus amount for vSphere services on the host. The host's shared memory may be larger than the amount of machine memory if memory is overcommitted (the aggregate virtual machine configured memory is much greater than machine memory). The value of this statistic reflects how effective transparent page sharing and memory overcommitment are for saving machine memory.|absolute|
|mem.unreserved.maximum|Amount of memory that is unreserved. Memory reservation not used by the Service Console, VMkernel, vSphere services and other powered on VMs’ user-specified memory reservations and overhead memory. This statistic is no longer relevant to virtual machine admission control, as reservations are now handled through resource pools.|absolute|
|mem.zero.minimum|Sum of zero metrics for all powered-on virtual machines, plus vSphere services on the host.|absolute|
|mem.llSwapIn.none|Amount of memory swapped-in from host cache|absolute|
|mem.swapused.minimum|Amount of memory that is used by swap. Sum of memory swapped of all powered on VMs and vSphere services on the host.|absolute|
|mem.sharedcommon.minimum|Amount of machine memory that is shared by all powered-on virtual machines and vSphere services on the host.Subtract this metric from the shared metric to gauge how much machine memory is saved due to sharing: shared - sharedcommon = machine memory (host memory) savings (KB)|absolute|
|mem.active.maximum|Sum of all active metrics for all powered-on virtual machines plus vSphere services (such as COS, vpxa) on the host.|absolute|
|mem.vmfs.pbc.workingSet.latest|Amount of file blocks whose addresses are cached in the VMFS PB Cache|absolute|
|mem.latency.average|Percentage of time the virtual machine is waiting to access swapped or compressed memory|absolute|
|mem.unreserved.minimum|Amount of memory that is unreserved. Memory reservation not used by the Service Console, VMkernel, vSphere services and other powered on VMs’ user-specified memory reservations and overhead memory. This statistic is no longer relevant to virtual machine admission control, as reservations are now handled through resource pools.|absolute|
|mem.vmfs.pbc.workingSetMax.latest|Maximum amount of file blocks whose addresses are cached in the VMFS PB Cache|absolute|
|mem.llSwapUsed.average|Space used for caching swapped pages in the host cache|absolute|
|mem.sharedcommon.maximum|Amount of machine memory that is shared by all powered-on virtual machines and vSphere services on the host.Subtract this metric from the shared metric to gauge how much machine memory is saved due to sharing:shared - sharedcommon = machine memory (host memory) savings (KB)|absolute|
|mem.swapout.none|Sum of swapout metrics from all powered-on virtual machines on the host.|absolute|
|mem.swapused.maximum|Amount of memory that is used by swap. Sum of memory swapped of all powered on VMs and vSphere services on the host.|absolute|
|mem.usage.none|Percentage of available machine memory:consumed ÷ machine-memory-size|absolute|
|mem.heapfree.minimum|Free address space in the VMkernel main heap.Varies based on number of physical devices and configuration options. There is no direct way for the user to increase or decrease this statistic. For informational purposes only: not useful for performance monitoring.|absolute|
|mem.heap.none|VMkernel virtual address space dedicated to VMkernel main heap and related data|absolute|
|mem.vmmemctl.average|The sum of all vmmemctl values for all powered-on virtual machines, plus vSphere services on the host. If the balloon target value is greater than the balloon value, the VMkernel inflates the balloon, causing more virtual machine memory to be reclaimed. If the balloon target value is less than the balloon value, the VMkernel deflates the balloon, which allows the virtual machine to consume additional memory if needed.|absolute|
|mem.sysUsage.maximum|Amount of host physical memory used by VMkernel for core functionality, such as device drivers and other internal uses. Does not include memory used by virtual machines or vSphere services.|absolute|
|mem.llSwapOutRate.average|Rate at which memory is being swapped from active memory to host cache|rate|
|mem.swapused.none|Amount of memory that is used by swap. Sum of memory swapped of all powered on VMs and vSphere services on the host.|absolute|
|mem.overhead.average|Total of all overhead metrics for powered-on virtual machines, plus the overhead of running vSphere services on the host.|absolute|
|mem.swapused.average|Amount of memory that is used by swap. Sum of memory swapped of all powered on VMs and vSphere services on the host.|absolute|
|mem.shared.maximum|Sum of all shared metrics for all powered-on virtual machines, plus amount for vSphere services on the host. The host's shared memory may be larger than the amount of machine memory if memory is overcommitted (the aggregate virtual machine configured memory is much greater than machine memory). The value of this statistic reflects how effective transparent page sharing and memory overcommitment are for saving machine memory.|absolute|
|mem.sharedcommon.none|Amount of machine memory that is shared by all powered-on virtual machines and vSphere services on the host.Subtract this metric from the shared metric to gauge how much machine memory is saved due to sharing: shared - sharedcommon = machine memory (host memory) savings (KB)|absolute|
|mem.usage.maximum|Percentage of available machine memory:consumed ÷ machine-memory-size|absolute|
- datastore

|COUNTER|DESCRIPTION|TYPE|
|---|---|---|
|datastore.datastoreWriteBytes.latest|Storage DRS datastore bytes written|absolute|
|datastore.datastoreIops.average|Average amount of time for an I/O operation to the datastore or LUN across all ESX hosts accessing it.|absolute|
|datastore.siocActiveTimePercentage.average|Percentage of time Storage I/O Control actively controlled datastore latency|absolute|
|datastore.numberWriteAveraged.average|Average number of write commands issued per second to the datastore during the collection interval|rate|
|datastore.totalWriteLatency.average|Average amount of time for a write operation to the datastore. Total latency = kernel latency + device latency.  |absolute|
|datastore.datastoreNormalWriteLatency.latest|Storage DRS datastore normalized write latency|absolute|
|datastore.sizeNormalizedDatastoreLatency.average|Storage I/O Control size-normalized I/O latency|absolute|
|datastore.datastoreVMObservedLatency.latest|The average datastore latency as seen by virtual machines|absolute|
|datastore.datastoreReadLoadMetric.latest|Storage DRS datastore metric for read workload model|absolute|
|datastore.datastoreMaxQueueDepth.latest|Storage I/O Control datastore maximum queue depth|absolute|
|datastore.maxTotalLatency.latest|Highest latency value across all datastores used by the host|absolute|
|datastore.datastoreReadIops.latest|Storage DRS datastore read I/O rate|absolute|
|datastore.datastoreReadBytes.latest|Storage DRS datastore bytes read|absolute|
|datastore.datastoreWriteIops.latest|Storage DRS datastore write I/O rate|absolute|
|datastore.datastoreWriteLoadMetric.latest|Storage DRS datastore metric for write workload model|absolute|
|datastore.datastoreWriteOIO.latest|Storage DRS datastore outstanding write requests|absolute| 
|datastore.datastoreReadOIO.latest|Storage DRS datastore outstanding read requests|absolute|
|datastore.numberReadAveraged.average|Average number of read commands issued per second to the datastore during the collection interval|rate|
|datastore.totalReadLatency.average|Average amount of time for a read operation from the datastore. Total latency = kernel latency + device latency.|absolute|
|datastore.read.average|Rate of reading data from the datastore|rate|
|datastore.datastoreNormalReadLatency.latest|Storage DRS datastore normalized read latency|absolute|
|datastore.write.average|Rate of writing data to the datastore|absolute|
