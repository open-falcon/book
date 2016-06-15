##Hardware monitoring

We have introduced the usual monitoring data source in section of Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The data collection of hardware can be done by HWCheck.

##HWCheck

Rvadmin hardware monitoring needs to install falcon-agent, only dell machines supported, and the monitoring index: CPU, memory, array card, magnetic disk, virtual disk, array card battery, BIOS, mainboard battery, fan, voltage, mainboard temperature, CPU temperature.

##Install:

1.Deploy dell official repo, install srvadmin and other dependecies.
You may also pack rpm to simplify the deployment.

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


##How to use

Parameter specification:

Direct execution hwcheck with no parameters will print out the detailed monitoring data by default.

```
hwcheck -d      # print metrics information, ie. data pushed to falcon-agent
        -p      # push data to falcon-agent
        -s      # set the value of STEP in push data，referring to monitoring frequency, 600s by default 
        -m      # single metric

        
```

##Deploy crontab

Deploy cron to detect on a regular basis, for example: 

```
cat /etc/cron.d/hwcheck
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/opt/dell/srvadmin/sbin:/opt/dell/srvadmin/bin
SHELL=/bin/bash

18 * * * * root /usr/bin/hwcheck -s 3600 -p >/dev/null 2>&1 &
```

referring to detecting per hour, the corresponding STEP value is set 3600.

##Configure alarm strategy in falcon-portal

The metric pushed to falcon-agent by hwcheck all begin with hw, such as hw.cpu_temp. Except for the actual temperature value, the value 0 in metric means fault, 1 warning, 2 OK. For example, deploy the following strategy in portal:

| metric/tags/note | condition | max | P |
| -- | -- | -- | -- |
| hw.bios [C1E/Cstate is not forbidden in BIOS] | all(#2)<2	 | 1 | 4 |
| hw.board_temp [Motherboard temperature is too high] | all(#3)>=35 | 1 | 4 |
| hw.cmos_bat [Motherboard battery has a problem] | all(#3)<2	 | 1 | 4 |
| hw.cpu [CPU possible faults] | all(#2)==1	 | 1 | 4 |
| hw.cpu [Major: CPU major fault] | all(#2)==0	 | 2 | 0 |
| hw.fan [fan failure] | all(#3)<2	 | 1 | 4 |
| hw.memory [Memory may be failure] | all(#1)==1	 | 1 | 4 |
| hw.memory [Major: major fault memory] | all(#1)==0	 | 2 | 0 |
| hw.pdisk [Major: magnetic disk major fault] | all(#1)==0	 | 2 | 0 |
| hw.raidcard [Array card warnings] | all(#2)==1 | 1 | 4 |
| hw.raidcard [Major: array card major fault] | all(#1)==0	 | 2 | 0 |
| hw.raidcard_bat [Array card battery warnings] | all(#2)==1	 | 1 | 4 |
| hw.raidcard_bat [Major: array card battery major fault] | all(#2)==0 | 2 | 0 |
| hw.vdisk [Disk array warnings] | all(#2)==1	 | 1 | 4 |
| hw.vdisk [Major: disk array major fault] | all(#2)==0	 | 2 | 0 |


