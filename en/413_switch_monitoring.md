##Switch monitoring

We have introduced the usual monitoring data source in section Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The running index of switch collection: we can write a script to collect each item of running index of switch by SNMP protocol, including memory usage, CPU usage, flow, ping delay and so on.

We can collect the monitoring index of switch directly by swcollector.

##Operating principle

Swcollector is a background program developed by Golang, designed by gaochao, and follow-up developed and maintained by freedomkk-qfeng at present.

Swcollector collects the running index of switch and report to Open-Falcon by SNMP protocol every certain period according to the IP list or IP segment of deployed switch in the configuration files.

The metric list collected:

* CPU utilization
* Memory utilization
* Ping delay (delay returned if normal, -1 returned if overtime, which can be used to survival warning)
* Net linking number (Cisco ASA is supported only at present when monitoring firewall)
* IfHCInOctets
* IfHCOutOctets
* IfHCInUcastPkts
* IfHCOutUcastPkts
* IfHCInBroadcastPkts
* IfHCOutBroadcastPkts
* IfHCInMulticastPkts
* IfHCOutMulticastPkts
* IfOperStatus (interface state，1 up, 2 down, 3 testing, 4 unknown, 5 dormant, 6 not Present, 7 lowerLayerDown)

The OID of CPU and memory may be different by the device manufacturers and OS version. The tested devices are as follows:

* Cisco IOS(Version 12)
* Cisco NX-OS(Version 6)
* Cisco IOS XR(Version 5)
* Cisco IOS XE(Version 15)
* Cisco ASA (Version 9)
* Ruijie 10G Routing Switch
* Huawei VRP(Version 8)
* Huawei VRP(Version 5.20)
* Huawei VRP(Version 5.120)
* Huawei VRP(Version 5.130)
* Huawei VRP(Version 5.70)
* Juniper JUNOS(Version 10)
* H3C(Version 5)
* H3C(Version 5.20)
* H3C(Version 7)

##Binary system installation

Download the latest binary system version from here. Notice: these binaries can only be run over 64bits Linux.

##Sound code installation

```
Dependent on $GOPATH/src/github.com/gaochao1/sw
cd $GOPATH/src/github.com/gaochao1/swcollector
go get ./...
./control build
./control pack
At the last step, a tar.gz will be packed, and we can use the packet to deploy service

Make sure to update sw first when upgrading.
cd $GOPATH/src/github.com/gaochao1/sw
git pull
```

##Deploy instruction
Swcollector need to be deployed to the server which has access authority to switch SNMP.

Ping detect using protogenetic ICMP protocol of Go. Swcollector needs root rights to run.


Some switch can be overtime while using protogenetic ICMP protocol of Go. And the temporary solution is to judge the unit type before the SNMP interface flow query. For part of such devices, snmpwalk command is called to collect data (some IOS XR of Huawei equipment and cisco), so it's better to install a snmpwalk command on the monitoring probe server.

##Configuration instruction

Configuration file to cfg.example.json, change the file name into cfg.json, and change the IP into the actual IP in this file.

Switch configuration instruction:

```
"switch":{
   "enabled": true,          
    "ipRange":[                        #P address section, for the valid IP in the section, ping packets are sent to detect, and SNMP request is sent to switch as for the survival IP.
       "192.168.1.0/24",      
       "192.168.56.102/32",
       "172.16.114.233" 
     ],
     "pingTimeout":300,                #Ping overtime, unit ms
    "pingRetry":4,                   #Ping detection retry times
    "community":"public",            #SNMP authentication string
    "snmpTimeout":2000,                #SNMP overtime, unit ms
    "snmpRetry":5,                    #SNMP retry times
    "ignoreIface": ["Nu","NU","Vlan","Vl","LoopBack"],    #neglectful interface, for example, the interface of *Nu* is Nu matching to ifName
    "ignorePkt": true,            #IfHCInUcastPkts and IfHCOutUcastPkts not collected
    "ignoreBroadcastPkt": true,   #IfHCInUcastPkts and IfHCOutUcastPkts not collected
    "ignoreMulticastPkt": true,   #IfHCInUcastPkts and IfHCOutUcastPkts not collected
    "ignoreOperStatus": true,     #IfOperStatus not collected
    "displayByBit": true,          #the unit of reported flow is bit when true, byte when false
    "fastPingMode": false,          #Whether open fastPing mode, the Ping is more efficient when open, and the scene that there will be a small probability Ping downtime switch address when the high concurrency can be solved, but fastPing may be filtered by firewall.
     "limitConcur": 1000           #limit the Concurrency SNMP 
}
```