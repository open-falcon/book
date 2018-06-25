<!-- toc -->

# Hardware Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of hareware can be done through [HWCheck](https://github.com/51web/hwcheck).

# HWCheck

Falcon-Agent needs to be installed, then modules like Rvadmin Tool can perform hareware monitor.

It only supports dell physical machines. The following metrics can be monitored:

cpu
memory
RAID
physical disk
virtual disk
RAID battery
BIOS
mainboard battery
fan
voltage
mainboard temperature
cpu temperature

# Installation Instruction

1. Configure the official repo of Dell and install dependencies like srvadmin and so on

```
#Reference: http://linux.dell.com/repo/hardware/latest/
wget -q -O - http://linux.dell.com/repo/hardware/latest/bootstrap.cgi | bash

yum install srvadmin-omacore srvadmin-omcommon srvadmin-storage-cli smbios-utils-bin lm_sensors dmidecode cronie
# Enable srvadmin service
/opt/dell/srvadmin/sbin/srvadmin-services.sh enable
/opt/dell/srvadmin/sbin/srvadmin-services.sh restart
# Configure lm-sensors
echo yes | /usr/sbin/sensors-detect
```

## You can also pack the rpm to simplify the deployment

```
git clone https://github.com/51web/hwcheck hwcheck-0.2
tar czf hwcheck-0.2.tar.gz hwcheck-0.2
rpmbuild -tb hwcheck-0.2.tar.gz
```


# How to Use

## Parameter Information

Execute hwcheck without parameters will directly print the detailed monitor data.

```
hwcheck -d      # print the metrics information，which means the data pushed to falcon-agent
        -p      # push data falcon-agent
        -s      # the value of STEP in data pushing, which means frequency; the default value is 600 seconds
        -m      # one specified metric
```

## Configure Crontab

Configure cron to test regularly, like：

```
cat /etc/cron.d/hwcheck
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/opt/dell/srvadmin/sbin:/opt/dell/srvadmin/bin
SHELL=/bin/bash

18 * * * * root /usr/bin/hwcheck -s 3600 -p >/dev/null 2>&1 &
```

The value of STEP is 3600, which means run a test each hour.


## Configure Alarm Strategy om Falcon-Portal

The metrics pushed by hwcheck  to falcon-agent all begin with hw, like hw.cpu_temp, except for the acutal number of the temperature.

In other value of metric: 0 means malfunction，1 means warning，2 meas OK. For example, configure the following strategies in Portal：

| metric/tags/note             | condition |   max |  P  |
|------------------------------ | --------- | ----- | --- |
| hw.bios [C1E in BIOS中/Cstate enabled] | all(#2)<2 | 1 | 4 |
| hw.board_temp [mainboard overheating] | all(#3)>=35 | 1 | 4 |
| hw.cmos_bat [mainboard bettery issue] | all(#3)<2 |1 | 4 |
| hw.cpu [CPU possible breakdown]         | all(#2)==1 | 1 | 4 |
| hw.cpu [Impoetant: CPU severe breakdown]   | all(#2)==0 | 2 | 0    |
| hw.fan [fan breakdown]        | all(#3)<2 | 1 | 4     |
| hw.memory [memory possible breakdown]     | all(#1)==1 | 1 | 4    |
| hw.memory [Important: memory severe breakdown] | all(#1)==0 | 2 | 0  |
| hw.pdisk [Important: physical disk severe breakdown] | all(#1)==0 | 2 | 0     |
| hw.raidcard [RAID warning] | all(#2)==1 | 1 | 4    |
| hw.raidcard [Important: RAID severe breakdown] | all(#1)==0 | 2 | 0  |
| hw.raidcard_bat [RAID bettery warning] | all(#2)==1 | 1 | 4   |
| hw.raidcard_bat [Important: RAID battery severe breakdown] | all(#2)==0 | 2 | 0  |
| hw.vdisk [disk array warning]    | all(#2)==1 | 1 | 4  |
| hw.vdisk [Important: disk array severe breakdown] | all(#2)==0 | 2 | 0   |


