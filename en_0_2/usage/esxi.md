<!-- toc -->

# ESXi Monitor

The operation system in the host machine of VMware ESXi. Open-Falcon cannot be installed in ESXi system, the monitor can be done in a normal way.

The data collection of system running index of the machine with ESXi operation system can be realized by writing scripts. The scripts can communicate with the switch about its running index, including memory usage, CPU usage, data traffic, disk usage and etc. [Esxicollector](https://github.com/humorless/esxicollector) is a script like this.。

## Working Principle

Esxicollector is a series of arranged scripts, designed and developed by [humorless](https://github.com/humorless/).

Esxicollector needs to be configured through cronjob. Users congigure the cronjob on a machine that can run cronjob, and write down the device that needs to be monitored and the Agent address of Open-Falcon that receives the monitor result. Esxicollector will collect the data of monitor index regularly according to the time interval set in cronjob, which is once a minute by default, and send them to Agent of Open-Falcon

List of collected metric:

* CPU usage

  `esxi.cpu.core`

* Total memory / usage percentage

  `esxi.cpu.memory.kliobytes.size`
  `esxi.cpu.memory.kliobytes.used`
  `esxi.cpu.memory.kliobytes.avail`

* Number of current process

  `esxi.current.process`

* Number of logged-in user

  `esxi.current.user`

* Number of virtual machine

  `esxi.current.vhost`

* Total disk space/usage percentage

  `esxi.df.size.kilobytes`
  `esxi.df.used.percentage`

* Disk error

  `esxi.disk.allocationfailure`

* Incoming/outgping data/packets of network card

  `esxi.net.in.octets`
  `esxi.net.in.ucast.pkts`
  `esxi.net.in.multicast.pkts`
  `esxi.net.in.broadcast.pkts`
  `esxi.net.out.octets`
  `esxi.net.out.ucast.pkts`
  `esxi.net.out.multicast.pkts`
  `esxi.net.out.broadcast.pkts`
	

## Installation

Download from [here](https://github.com/humorless/esxicollector).

  1. Install SNMP Command

  `yum -y install net-snmp net-snmp-utils`

  2. Download the file of VMware ESXi MIB and copy then to the folder `/usr/share/snmp/mibs`

  3. Configure SNMP Environment

  `mkdir ~/.snmp`  
  `echo "mibs +ALL" > ~/.snmp/snmp.conf`

  4. Fill in proper parameter in `esxi_collector.sh`

  5. Configure Cronjobs 
  
  ` * * * * * esxi_collector.sh `


## Extend and Develop New Monitor Index

The script ```snmp_queries.sh``` can call out basic snmp commands and output the result of snmp after execution. Users can develop new script by comparing the results of executing ```60_esxi_*.sh```.
