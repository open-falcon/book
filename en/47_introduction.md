##Introduction

Falcon-agent can collect more than 200 monitor index data automatically without configuration, for example any data related to cpu, internal storage, disk io, network card, etc., can be detected and collected automatically.

##Port monitor

In the initial compiling stages of falcon-agent, all ports monitored by the machine should be submitted to server, for instance the machine monitors 3 ports of 80, 443 and 22, then 3 data will be submitted automatically: 

```
net.port.listen/port=22
net.port.listen/port=80
net.port.listen/port=443
```

The submitted port data, value is 1, if later certain ports no longer monitor, then the data submission will be stop immediately. For whether it is okay to do this, there are two questions: 

* There are perhaps many ports monitored by machines, but maybe there are not many ports really need to be monitored, which is a waste of resources.
* Currently Open-Falcon does not support nodata monitor. If the port crashed and cannot submit data, it cannot be detected without nodata system.

Improvement should be made.

Which ports should be collected by agent is automatically calculated by the strategy of user configuration. Because no matter what, the monitor configuration strategy can not be dismissed. For instance user configures 2 ports: 

```
net.port.listen/port=8080 if all(#3) == 0 then alarm()
net.port.listen/port=8081 if all(#3) == 0 then alarm()
```
If the strategy is bound to certain HostGroup, then the machines under the HostGroup will collect the data of ports 8080 and 8081. The information is sent through the coordination mechanism of agent and hbs.

Agent captures which ports are currently monitoring through ```ss -tln```, if 8080is monitoring, set value=1, submit to transfer, if 8081 is not monitoring, set value=0, submit to transfer. 

##Process monitor

Process monitor is similar to port monitor. It also automatically calculates to collect the information of which process and submit through user configuration strategy. For example:

```
proc.num/name=ntpd if all(#2) == 0 then alarm()
proc.num/name=crond if all(#2) == 0 then alarm()
proc.num/cmdline=uic.properties if all(#2) == 0 then alarm()
```

Proc.num indicates the number of processes, for instance actually there are multiple processes can be named as crond. It suports two kinds of tag configuration, one is process name, the other is configuration process cmdline, but they cannot appear at the same time. 

Then now DEV writes a program, how do I know the process name? Firstly, I should get the process ID, cat /proc/$pid/status, then I can find the name letters in the ID name. falcon-agent is collected based on the name field. There can be a possible bug. The name field should be no more than 15 letters. So if your process name is quite long, maybe it will be cut off. We ignore the original process name before it is cut off. Agent will use the name in the status file. Therefore, when you configure the name tag, you should take a look at the status file and obtain name in it, instead of creating a process name you think is right.

Then for cmdline, its name is collected from the file /proc/$pid/status, the name of cmdline is collected from the file /proc/$pid/cmdline. Inside the file stored the command you will use for starting the process. For example, you use java -c uic.properties to start a Java process, the process is named as java. Actually the name for all java processes is java, so we cannot distinguish them through name field. What should we do? It is the time to turn to the content of the file /proc/$pid/cmdline.

The content of cmdline is your start command, which is not accurate because all the spacing are missing. Actually the spacing is automatically replaced by \0. Do not worry, just click with the mouse, copy and paste. Do Not manually add spacing configuration in the strategy. Spacing in the monitor strategy tag is not allowed.
In the above examples, the content of java -c uic.properties in cmdline will become: java-cuic.properties, and there is no need to copy and configure the whole cmdline in the strategy. Although to name the tag is all matched, i.e. use == to compare name, cmdline is not. We only need to copy certain part of cmdline character string which can be different from other process names. For instance the above configuration:
```
proc.num/cmdline=uic.properties if all(#2) == 0 then alarm()
```

It is OK. When falcon-agent gets the content of cmdline file, it will use the method of strings.Contains() to decide.

Does it sound super complicated? Absolutely not, if there is a port in your process is monitoring, it is OK to configure one port monitoring. There is no need to configure port monitor and process monitor at the same time. After all, if the process crashed, the port will not be monitoring as well.

