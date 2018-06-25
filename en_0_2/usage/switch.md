<!-- toc -->

# Switch Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The running data collection of switch, like memory usage, Cpu usage, data traffic, ping and etc., can be done through a script in SNMP protocal.

The monitor data collection of switch can be done through [swcollector](https://github.com/gaochao1/swcollector).



## Working Principle

Swcollector is a background program in Golang developed by [gaochao](https://github.com/gaochao1) and currently further developed and maintained by [freedomkk-qfeng](https://github.com/freedomkk-qfeng).

Swcollector collects the running data of switch and pushes them to Open-Falcon in a fixed time cycle according to the switch IP list or IP network segment in configuration file.

List of collected metrics:

* Percentage of CPU use
* Percentage of memory
* Ping（responds -1 when timeout for survival alarm）
* IfHCInOctets
* IfHCOutOctets
* IfHCInUcastPkts
* IfHCOutUcastPkts
* IfHCInBroadcastPkts
* IfHCOutBroadcastPkts
* IfHCInMulticastPkts
* IfHCOutMulticastPkts
* IfInDiscards
* IfOutDiscards
* IfInErrors
* IfOutErros
* IfInUnknownProtos
* IfOutQLen
* IfSpeed
* IfSpeedPercent 
* IfOperStatus (meaning of number: 1 up, 2 down, 3 testing, 4 unknown, 5 dormant, 6 notPresent, 7 lowerLayerDown)
	

The OID of CPU and memory is private. It may differ due to different manufacturers and verions of operation system. Here are the devices that have been tested:

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
* DELL

#### Binary Installation
Download the latest compiled installation pack from [here](https://github.com/gaochao1/swcollector/releases). Attention: it it only supported in 64-bit Linux system.

#### Source Code Installation
```
	Depends on $GOPATH/src/github.com/gaochao1/sw
	cd $GOPATH/src/github.com/gaochao1/swcollector
	go get ./...
	chmod +x control
	./control build
	./control pack
	The last step will generate an installation tar.gz for service deployment.
	
	Make sure sw is updated before update
	cd $GOPATH/src/github.com/gaochao1/sw
	git pull
```

#### Deployment Instruction

Swcollector needs to be deployed on the server with the access permission to switch SNMP.

Swcollector needs root authority when detects Ping through ICMP protocal of native Go.

Gosnmp and snmpwalk are supported in data collection. Snmpwalk command needs to be installed in the probe monitor server in snmpwalk mode.

#### Configuration Information

Rename the congifuration file cfg,json as the standard of cfg.example.json and change the IP in it to the actual IP you use.
Configuration information：
```
{
    "debug": true,
	"debugmetric":{                                          # specific debug metric in the log
		"endpoints":["192.168.56.101","192.168.56.102"],     # required
		"metrics":["switch.if.In","switch.if.Out"],          # required
		"tags":"ifName=Fa0/1"  # adapt the tag if there are any; if "" then print all the information of the metric
	},
	"switch":{
	   "enabled": true,
		"ipRange":[            #IP range of this switch; send a ping packet testing the validity of this network segment and send IP SNMP request to the surviving IP 
            "192.168.56.101/32",      
            "192.168.56.102-192.168.56.120",#the format is supported; testing the validity of this IP range and sending IP SNMP request to the surviving IP 
            "172.16.114.233" 
 		],
		"gosnmp":true,         #collecting through gosnmp; if false then snmpwalk
 		"pingTimeout":300,     #Ping timeout measured in millisecond
		"pingRetry":4,         #Ping retry counter
		"community":"public",  #SNMP authentication strings
		"snmpTimeout":1000,    #SNMP timeout measured in millisecond
		"snmpRetry":5,         #SNMP retry counter
		"ignoreIface": ["Nu","NU","Vlan","Vl"], #ignored port; if Nu then adapt the port of "if Name is *Nu*"
		"ignoreOperStatus": true,               #ignore OperStatus
		"speedlimit":0,  #Speed upper limit; if the collected and calculated speed exceeds it then ingore sending; if it is set 0; the upper speed limit is the speed of the port; virtual interface like vlan without ifSpeed cannot not in the comparison of the maximun
		"ignorePkt": true,                      #ignore IfHCInUcastPkts and IfHCOutUcastPkts
		"pktlimit": 0,  #Packet number upper limit; if the collected and calculated packet number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum
		"ignoreBroadcastPkt": true,  #ingore IfHCInBroadcastPkts and IfHCOutBroadcastPkts
		"broadcastPktlimit": 0,  #broadcastPkt number upper limit; if the collected and calculated broadcastPkt number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum 
		"ignoreMulticastPkt": true,  #ignore IfHCInMulticastPkts and IfHCOutMulticastPkts
		"multicastPktlimit": 0,  #multicastPkt number upper limit; if the collected and calculated multicastPkt number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum 
		"ignoreDiscards": true,   #ignore IfInDiscards and IfOutDiscards
		"discardsPktlimit": 0,   #discardsPkt number upper limit; if the collected and calculated discardsPkt number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum
		"ignoreErrors": true,   #ignore IfInErrors and IfOutErros
		"errorsPktlimit": 0,    #errorsPkt number upper limit; if the collected and calculated errorsPkt number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum
		"ignoreUnknownProtos":true,    #ignore IfInUnknownProtos
		"unknownProtosPktlimit": 0,    #unknownProtosPkt number upper limit; if the collected and calculated unknownProtosPkt number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum
		"ignoreOutQLen":true,    #ignore IfOutQLen
		"outQLenPktlimit": 0,   #outQLenPkt number upper limit; if the collected and calculated outQLenPkt number exceeds it then ingore sending it; if it is set 0 then it is out of the comparison of maximum
		"fastPingMode": true,  
		"limitConcur": 1000, #concurrence system of switch collection
		"limitCon": 4 #number limit of how many metric collections concur on a single switch
 	}, 
	"switchhosts":{
		"enabled":true,
		"hosts":"./hosts.json"  #list of custom host and its IP; if it is configured, then the host here will replace the IP address during sending
	},
	"customMetrics":{         
		"enabled":true,
		"template":"./custom.json"    #custom list of custom metric; if it is configured, the extra custom metrics will be collected according to this configuration
	},
    "transfer": {
        "enabled": true,
        "addr": "127.0.0.1:8433",
        "interval": 60,
        "timeout": 1000
    },
    "http": {
        "enabled": true,
        "listen": ":1989"
    }
}

```
Custom Host Configuration
```
{
	"hosts":
	{
		"192.168.160":"test1",
		"192.168.88.161":"test2",
		"192.168.33.2":"test3",
		"192.168.31.51":"test4"
	}
}
```
Custom OID Configuration

```
{
	"metrics":
		[
			{
			"ipRange":[
         	   "192.168.0.1-192.168.0.2", #the addess collected by this custom OID
			   "192.168.1.1"
 			],
			"metric":"switch.AnyconnectSession", #custom metric
			"tag":"",          #custom tag
			"type":"GAUGE",
			"oid":"1.3.6.1.4.1.9.9.392.1.3.35.0" #custom oid
			},
			#Custom OID only supports snmp get collectioin, so please not leave out any OID information; snmpwalk test is recommended
			#the number of online anyconnect in cisco asa 上
			{
			"ipRange":[
         	   "192.168.1.160-192.168.1.161"
 			],
			"metric":"switch.ConnectionStat",
			"tag":"",
			"type":"GAUGE",
			"oid":"1.3.6.1.4.1.9.9.147.1.2.2.2.1.5.40.6"
			},
			#this is the number of firewall connection in cisco asa
			{
			"ipRange":[
         	   "192.168.33.2"
 			],
			"metric":"switch.TempStatus",
			"tag":"",
			"type":"GAUGE",
			"oid":"1.3.6.1.4.1.9.9.13.1.3.1.3.1004"
			}
			#the temperature of cisco switch; please notice that the universal oid is "1.3.6.1.4.1.9.9.13.1.3.1.3", in which 1004 is hareware index; box-type switches may have several temperatures (several line cards) so please fill in the spefific OID and the corresponding tag according to actual situation
		]
}
```

#### Deployment Information
Due to the character of concurrent collection, the time consuming of each collecting cycle depends on the switch with the slowest collecting speed.
We can view the time consuming of each switch in debug mode.
```
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.10.1 PingResult: true len_list: 440 UsedTime: 5
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.10.252 PingResult: true len_list: 97 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.13.1 PingResult: true len_list: 24 UsedTime: 1
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.14.1 PingResult: true len_list: 23 UsedTime: 1
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.23.1 PingResult: true len_list: 61 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.1 PingResult: true len_list: 55 UsedTime: 1
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.5 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.6 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.11 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.12 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.13 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.14 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.15 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.16 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.17 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.18 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.19 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.20 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.21 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.22 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.23 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.24 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:24 swifstat.go:121: IP: 192.168.12.25 PingResult: true len_list: 26 UsedTime: 2
2016/08/16 21:31:29 swifstat.go:121: IP: 192.168.11.2 PingResult: true len_list: 348 UsedTime: 10
2016/08/16 21:31:29 swifstat.go:177: UpdateIfStats complete. Process time 10.700895998s.
```
We can see that the switch 192.168.11.2 uses 10 seconds, which is the longest time that takes up the most time in the collecting cycle.

We can do some categorizing work of switches according to these information. Switches that respond quickly to snmp can be collected by one swcollector. The timeout setting of this group can be shorter and so is the collecting cycle, like 30 seconds.
Switches that respond slowly to snmp can be collected by another swcollector. The timeout setting of this group can be longer and so is the collecting cycle, like 5 minutes.

Responding snmp messages consumes cpu, so switch has a timeout limit of responding snmp messages to some extent. When CPU is pretty high, the switch can even ignore requests with lower priority like snmp and icmp.

Different brand has a different limit to this issue and the limit of some model can be change in configuration. User can also loosen up the snmp speed limit of the switch to raise the speed of responding snmp message if allowed.

After this configuration, the length of the "plank" in the "barrel" has been adjusted and the appropriate collecting cycle has been chosen. A single swcollector instance can carry millions of switches or even more

#### Information of v3-v4 Update（Important）
Since the swcollector of v4 changed the port data sending method from COUNTER to GAUGE, the data in Graph will **all be lost**! 
Therefore, we recommend users to enable the custom host feature and change the IP address of the switch to a new endpoint through this feature. For example, the original Ip is
```
			"ipRange":[
         	   "192.168.0.1-192.168.0.2",
			   "192.168.1.1"
 			],
```
Enable switchhosts.enabled = true and configure hosts.json
```
{
	"hosts":
	{
		"192.168.0.1":"sw-192.168.0.1",
		"192.168.0.2":"sw-192.168.0.1",
		"192.168.1.1":"sw-192.168.1.1"
	}
}
```
Then the endpoint will be sent as a new one "sw-192.168.x.x" and the original endpoint is saved as history.