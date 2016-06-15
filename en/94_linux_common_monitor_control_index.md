# 9.4 Linux common monitor control index

##Linux operation and maintenance basic collection item

As for operation and maintenance, it’s not serious to have problems, but to fail to catch the scene so that we may get into trouble faced with the problem. Therefore, it is of great significance to rely on a strong monitoring system and collect as many indicators as possible. But which indicators are meaningful? In the thought from practice, the experience summed up by the engineers in long fought is the most valuable.

In the long-term work practice of operation and maintenance engineers, we summarize a number of indicators often referred to in the system operation and maintenance process, including the following categories:

* CPU
* Load
* Memory
* Disk
* IO
* About network
* Kernel core parameter
* ss statistics output
* port collection
* Course survival information collection of core service
* Key business course resource consumption
* NTP offset collection
* DNS resolution collection
* 
As for every category, the specific concrete indicators are as follows. These indicators are all directly supported by the agent component of open-falcon. Falcon-agent collects the relevant indicators for every certain period (60s at present) and report to server.

###CPU related collection item 
Calculation method: got by collecting / proc / stat. You can refer to the statistical output of the sar command to understand.

###Disk related collection item 

Calculation method: firstly, read / proc / mounts to get all the mount points, then get the service condition of blocks and inode by syscall.Statfs_t. Each metric will be attached with a group of tag description like mount = $ mount, fstype = $ fstype, wherein $ mount is the mount point just like / home, $ fstype is a file system such as ext4.

* df.bytes.free: disk available quantity, int64
* df.bytes.free.percent: the percentage that the disk available quantity occupied the total, float64, such as 32.1
* df.bytes.total: disk total quantity, int64
* df.bytes.used: used disk quantity, int64
* df.bytes.used.percent: the percentage that the used disk quantity occupied the total, float64
* df.inodes.total: inode quantity, int64
* df.inodes.free: available inode quatity, int64
* df.inodes.free.percent: percentage of available inode, float64
* df.inodes.used: used inode data, int64
* df.inodes.used.percent: percentage of used inode, float64

###megacli tool output

Use megacli tool to read RAID information and each metric will be attached with a group of tag description to indicate the PD or VD it belongs to. The PD format is PD = Enclosure_ID: SLOT_ID, such as PD = 32: 0 indicating the first disk, VD = 0 indicating the first logical disk.

* sys.disk.lsiraid.pd.Media_Error_Count: this indicator as well as the following three are used as data collection at present and it doesn’t mean disk damage(only indicating the probability of disk damage increase) 
* sys.disk.lsiraid.pd.Other_Error_Count
* sys.disk.lsiraid.pd.Predictive_Failure_Count
* sys.disk.lsiraid.pd.Drive_Temperature
* sys.disk.lsiraid.pd.Firmware_state: if it is not 0, the physical disk goes wrong
* sys.disk.lsiraid.vd.cache_policy: if it is not 0, the cache strategy of this logical disk is not corresponding to the settings
* sys.disk.lsiraid.vd.state:  if it is not 0, this logical disk goes wrong

###SMART tool output

Use smartctl tool to read SMART information. All indicators are used as data collection at present and it doesn’t mean disk damage (only indicating the probability of disk damage increases). Each metric will be attached with a group of tag description to indicate the drive, such as device = / dev / sda .

* sys.disk.smart.Reallocated_Sector_Ct
* sys.disk.smart.Spin_Retry_Count
* sys.disk.smart.Reallocated_Event_Count
* sys.disk.smart.Current_Pending_Sector
* sys.disk.smart.Offline_Uncorrectable
* sys.disk.smart.Temperature_Celsius

###Partition read-write monitoring

Test whether all the mount partition is able to read and write. Each metric will have a group of tag description to show mount points, such as mount=/home

* sys.disk.rw if it is not 0, the read and write of this partition goes wrong

###IO Related Collection Items 
Calculation method: Collect/proc/diskstats once every second, calculate the difference, which are all counter types. Every metric has a group of tag description, like device=$device, which represents specific equipment, such as sda1 and sdb. Users can understand the specific meaning of metric referring to the help document of iostat.

* disk.io.ios_in_progress: Number of actual I/O requests currently in flight.
* disk.io.msec_read: Total number of ms spent by all reads.
* disk.io.msec_total: Amount of time during which ios_in_progress >= 1.
* disk.io.msec_weighted_total: Measure of recent I/O completion time and backlog.
* disk.io.msec_write: Total number of ms spent by all writes.
* disk.io.read_merged: Adjacent read requests merged in a single req.
* disk.io.read_requests: Total number of reads completed successfully.
* disk.io.read_sectors: Total number of sectors read successfully.
* disk.io.write_merged: Adjacent write requests merged in a single req.
* disk.io.write_requests: Total number of writes completed successfully.
* disk.io.write_sectors: Total number of sectors written successfully.
* disk.io.read_bytes: Numbers in bytes
* disk.io.write_bytes: Numbers in bytes
* disk.io.avgrq_sz: The values below are shown while iostat -x 1
* disk.io.avgqu-sz
* disk.io.await
* disk.io.svctm
* disk.io.util: A percentage, for example 56.43 means 56.43%

###Machine Loading Related Collection Items

Calculation method: read /proc/loadavg, which are all original value types:

* load.1min
* load.5min
* load.15min

###Internal Storage Related Collection Items

Calculation method: read the content in /proc/meminfo, among which mem.memfree is free+buffers+cached, mem.memused=mem.memtotal-mem.memfree. Users can specifically understand the meaning of every metric referring to the output and help document of free command. 

* mem.memtotal: Total size of internal storage
* mem.memused: The amount of internal storage used
* mem.memused.percent: Proportion of internal storage used
* mem.memfree
* mem.memfree.percent
* mem.swaptotal: Total size of swap
* Mem.swapused: The amount of swap used
* Mem.swapused.percent: Proportion of swap used
* mem.swapfree
* mem.swapfree.percent

###Network Related Collection Items

Calculation method: read the content of /proc/net/dev, every metric has an extra group of tags, such as iface=$iface, and indicates the specific interface, e.g. eth0. The metric with in means the situation of coming in, and the one with out means the the situation of going in, total means the total of in+out, the metrics below are supported: 

* net.if.in.bytes
* net.if.in.compressed
* net.if.in.dropped
* net.if.in.errors
* net.if.in.fifo.errs
* net.if.in.frame.errs
* net.if.in.multicast
* net.if.in.packets
* net.if.out.bytes
* net.if.out.carrier.errs
* net.if.out.collisions
* net.if.out.compressed
* net.if.out.dropped
* net.if.out.errors
* net.if.out.fifo.errs
* net.if.out.packets
* net.if.total.bytes
* net.if.total.dropped
* net.if.total.errors
* net.if.total.packets

###Port Collection Items
Calculation method: determine whether the specified port is in the state of listen through ss -ln. For the original value type, the value is either 1: which means listening, or 0, which means not listening. Every metric has an extra group of tags, such as port=$port, $port is the specific port.

* net.port.listen

###Machine Kernel Configuration

* Kernel.maxfiles: Read /proc/sys/fs/file-max
* Kernel.files.allocated: Read the first Field of /proc/sys/fs/file-nr
* Kernel.files.left: Value=kernel.maxfiles-kernel.files.allocated
* Kernel.maxproc: Read /proc/sys/kernel/pid_max

###ntp Collection Items

We use ntpq -pn to obtain the offset whose local time is relative to ntp server. 

* Sys.ntp.offset: The offset time of the machine in ms, when the value is too large or 0 means it is abnormal, and it will alarm

###Process Monitor
* proc.num: Numbers of certain process are determined through two types of scenarios, one is to determine by the name of the process, such as name=sshd; the other is to determine by cmdline, for example the Java application process names are possibly all java, if we cannot tell through the first scenario, then we can configure cmdline, like cmdline=./falcon_agent-c./cfg.ini

###Process Resource Monitor
* process.cpu.all: Process and its sub process use cpu of sys+user, the unit is jiffies
* process.cpu.sys: Process and its sub process use sys cpu, the unit is jiffies
* process.cpu.user: Process and its sub process use user cpu, the unit is jiffies
* process.swap: Process and its sub process use swap, the unit is page
* process.fd: Numbers of document descriptors used by process
* process.mem: internal storage occupied by process, the unit is byte

###ss Command Output
* ss.orphaned
* ss.closed
* ss.timewait
* ss.slabinfo.timewait
* ss.synrecv
* Ss.estab