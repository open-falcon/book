<!-- toc -->

# Basic collection item of Linux Operation

Operation personnel are never afraid of errors but the lack of solution due to incomprehensive information. Therefore, it is very meaningful to collect as many indexes as possible relying on powerful monitor system. But what index is meaningful? Guided by the motto that experience comes from practice, the engineers have learned the lessons.

In the engineers' long-term working practice, we have summed up some indexes that we frequently refer to. They are:

- CPU
- Load
- Memory
- Hard Drive
- IO
- Network-related data
- Kernel parameter
- ss statistics output
- Port collection
- Survival collection of key service process
- Resource consumption of key business process
- NTP offset collection
- DNS resolution collection

The detailed indexes of each category are shown below, which are supported by the Agent module of Open-Falcon. Agent will collect the data of those indexes in certain time interval (currently 60 seconds) and report them to the server.

# CPU-Related Index

Calculation method: collecting the data of /proc/stat, similar to statistics output  of SAR command

- cpu.idle：Percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request.
- cpu.busy：complementation of cpu.idle that equals 100 minus cpu.idle
- cpu.guest：Percentage of time spent by the CPU or CPUs to run a virtual processor.
- cpu.iowait：Percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request.
- cpu.irq：Percentage of time spent by the CPU or CPUs to service hardware interrupts.
- cpu.softirq：Percentage of time spent by the CPU or CPUs to service software interrupts.
- cpu.nice：Percentage of CPU utilization that occurred while executing at the user level with nice priority.
- cpu.steal：Percentage of time spent in involuntary wait by the virtual CPU or CPUs while the hypervisor was servicing another virtual processor.
- cpu.system：Percentage of CPU utilization that occurred while executing at the system level (kernel).
- cpu.user：Percentage of CPU utilization that occurred while executing at the user level (application).
- cpu.cnt：Number of cores of cpu.
- cpu.switches：A kind of counter that counts how many times CPU switches between context.


# Disk-related Index

Calculation method: retrieve all the mount points after reading /proc/mounts, then retrieving the usage information of blocks and inode from syscall.Statfs_t. Each metric comes with a couple of tags, like "mount=$mount,fstype=$fstype". "$mount" is the mount point like "/home" and the "$fstype" the file system like ext4.

- df.bytes.free: the available capacity of the disk, int64
- df.bytes.free.percent: the percentage of the available capacity of the disk float64, like 32.1
- df.bytes.total: the total capacity of the disk, int64
- df.bytes.used: the used capacity of the disk, int64
- df.bytes.used.percent: the percentage of the used capacity of the disk, float64
- df.inodes.total: the number of inodes, int64
- df.inodes.free: the number of available inodes, int64
- df.inodes.free.percent: the percentage of available inodes, float64
- df.inodes.used: the number of used inodes, int64
- df.inodes.used.percent: the percentage of used inodes, float64
  
# Megacli Tool Output
Read RAID information by using megacli tool. Each metric comes with a couple of tags that mark which PD or VD it belongs to. PD are in form of "PD=Enclosure_ID:SLOT_ID". For example, "PD=32:0" marks the first disk and "VD=0" marks the first logical disk.

- sys.disk.lsiraid.pd.Media_Error_Count
- sys.disk.lsiraid.pd.Other_Error_Count
- sys.disk.lsiraid.pd.Predictive_Failure_Count
- sys.disk.lsiraid.pd.Drive_Temperature
(The four indexes above are only for data collection. It does mean that the disk is damaged. However, It do signify that the possibility of disk damage has increased.)
- sys.disk.lsiraid.pd.Firmware_state: this physical disk is damaged when the value is not 0
- sys.disk.lsiraid.vd.cache_policy: the cache policy of this logical disk and the configuration are not corresponding when the value is not 0
- sys.disk.lsiraid.vd.state: this physical disk is damaged when the value is not 0

# SMART Tool Output
Read SMART information by using smartctl tool. Currently all indexes above are only for data collection. They does mean that the disk is damaged. However, they signify that the possibility of disk damage has increased. Each metric comes with a tag that marks the drive letter, like device=/dev/sda。

- sys.disk.smart.Reallocated_Sector_Ct
- sys.disk.smart.Spin_Retry_Count
- sys.disk.smart.Reallocated_Event_Count
- sys.disk.smart.Current_Pending_Sector
- sys.disk.smart.Offline_Uncorrectable
- sys.disk.smart.Temperature_Celsius

# Partition Read-Write Monitor
It tests whether all the mounted partitions are able to read and write. Each metric comes with a tag that marks the mount point, like "mount=/home"

- sys.disk.rw： an error occurs in the reading and writing process of this partition when the value is not 0

# IO-Related Index
Calculation method: collect the data of /proc/diskstats and calculate the delta like a counter. Each metric comes with a tag, like "device=$device", that marks a specific device, like sda1 or sdb. Please consult the help files of iostat for the meaning of specific metrics.

- disk.io.ios_in_progress：Number of actual I/O requests currently in flight.
- disk.io.msec_read：Total number of ms spent by all reads.
- disk.io.msec_total：Amount of time during which ios_in_progress >= 1.
- disk.io.msec_weighted_total：Measure of recent I/O completion time and backlog.
- disk.io.msec_write：Total number of ms spent by all writes.
- disk.io.read_merged：Adjacent read requests merged in a single req.
- disk.io.read_requests：Total number of reads completed successfully.
- disk.io.read_sectors：Total number of sectors read successfully.
- disk.io.write_merged：Adjacent write requests merged in a single req.
- disk.io.write_requests：total number of writes completed successfully.
- disk.io.write_sectors：total number of sectors written successfully.
- disk.io.read_bytes：the number in the result measured in byte
- disk.io.write_bytes：the number in the result measured in byte
- disk.io.avgrq_sz：the numbers below are what iostat -x 1 sees
- disk.io.avgqu-sz 
- disk.io.await
- disk.io.svctm
- disk.io.util：the number in percentage, like 56.43 in 56.43%


# Load-Related Index

Calculation method: just read the original value of /proc/loadavg

- load.1min
- load.5min
- load.15min

# Memory-Related Index

Calculation method: just read the content of "/proc/meminfo", in which "mem.memfree" is free+buffers+cachedand "mem.memused" is "mem.memtotal-mem.memfree". Please consult the help files of the free command and its output for the meaning of each metric.

- mem.memtotal：the size of total memory
- mem.memused：the size of used memory
- mem.memused.percent：the percentage of the size of used memory
- mem.memfree 
- mem.memfree.percent
- mem.swaptotal：the size of total swap
- mem.swapused：the size of used swap
- mem.swapused.percent：the percentage of the size of used swap
- mem.swapfree 
- mem.swapfree.percent


# Network-Related Index

Calculation method: just read the content of "/proc/net/dev". Each metric comes with a tag which marks the specific interface. For example, "iface=$iface" marks eth0. The metric with in indicates input status and out indicates output status. Total covers the input and output. Here are the supported metrics:

- net.if.in.bytes
- net.if.in.compressed
- net.if.in.dropped
- net.if.in.errors
- net.if.in.fifo.errs
- net.if.in.frame.errs
- net.if.in.multicast
- net.if.in.packets
- net.if.out.bytes
- net.if.out.carrier.errs
- net.if.out.collisions
- net.if.out.compressed
- net.if.out.dropped
- net.if.out.errors
- net.if.out.fifo.errs
- net.if.out.packets
- net.if.total.bytes
- net.if.total.dropped
- net.if.total.errors
- net.if.total.packets

# Port Index

Calculation method: check whether a specific port is in listening status through ss -ln. The original value is 1 or 0. 1 means it is in listening status and 0 means not. Each metric comes with a tag, like "port=$port" in which "$port" is the specific port.

- net.port.listen

# Kernel Index

- kernel.maxfiles： read /proc/sys/fs/file-max
- kernel.files.allocated：read the first field of /proc/sys/fs/file-nr
- kernel.files.left：the value is "kernel.maxfiles-kernel.files.allocated"
- kernel.maxproc：read /proc/sys/kernel/pid_max

# Ntp Index

Retrieve the time offset of this machine for the time of NTP server.

- sys.ntp.offset： The time offset of this machine measured in millisecond. It is necessary to send an alarm when the value is 0 or quite high.

# Process Monitor进程监控

- proc.num：Identify the number of certain process. Here are two methods: 
 - 1. Identify from the name of a process, like name=sshd.
 - 2. Identify from the cmdline.

For example, the name of processes in Java is java，so the first method will not work out, then we can configure cmdline, like cmdline=./falcon_agent-c./cfg.ini


# Process Resource Monitor

- process.cpu.all: the sys CPU and user CPU that a process and its subprocess use, measured in jiffies
- process.cpu.sys: the sys CPU that a process and its subprocess use, measured in jiffies
- process.cpu.user: the user CPU that a process and its subprocess use, measured in jiffies
- process.swap: the swap that a process and its subprocess use, measured in page
- process.fd: the number of file descriptors that a process uses进程使用的文件描述符个数
- process.mem: the memory that a process uses, measured in byte

# SS Command Output

- ss.orphaned
- ss.closed
- ss.timewait
- ss.slabinfo.timewait
- ss.synrecv
- ss.estab

