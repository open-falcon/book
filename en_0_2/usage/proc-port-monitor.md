<!-- toc -->

# Introduction

We say that Falcon-agent will automatically find and collect the data of over 200 metrics about CPU, memory, diskio, network card and etc.

# Port Monitor

In the first place of development, Falcon-agent is designed to push the all the port monitor data of this machine to server. For example, if the machine monitors 80, 443, 22 thre ports, it will sends three pieces of data:

```
net.port.listen/port=22
net.port.listen/port=80
net.port.listen/port=443
```

The value of them are all 1. Then the data of a port stops from being pushed when the machine does not monitor it anymore. Is that OK? There are two questions

- The machine monitors a lot of ports but it does not want to monitor all of them, which causes the waste of resources.
- Currently, Open-falcon has supported nodata monitoring to detect port survival through the nodata mechanism.

So we improved it.

Which port that Agent needs to collect is calculated and decided by user's strategy configuration. Because no matter how, user cannot skip configuring monitor strategy。For example, if a user configures two ports:

```
net.port.listen/port=8080 if all(#3) == 0 then alarm()
net.port.listen/port=8081 if all(#3) == 0 then alarm()
```

Bind this strategy to a HostGroup and the machine in this HostGroup will collect port 8080 and 8081. This command is sent through Agent and the Heartbeat system in HBS.

Agent knows currently which ports are being monitored from `ss -tln`. If 8080 is being monitored, it sets the value to 1 and pushes to Transfer; if not, it sets the value to 0 and pushes it to Transfer.

# Process Monitor

Process monitor is similar to Port monitor. Which process that Agent needs to collect is calculated and decided by user's strategy configuration. Here is an example:

```
proc.num/name=ntpd if all(#2) == 0 then alarm()
proc.num/name=crond if all(#2) == 0 then alarm()
proc.num/cmdline=uic.properties if all(#2) == 0 then alarm()
```

Proc.num means the number of process. There may be more than one process whose name is crond. The process name and process cmdline are supported in tag configuration, but they cannot both exist at the same time.

How do I know the process name of the program from developers?

First, we need the Process ID. `Cat /proc/$pid/status`. Falcon-Agent collects the name according to the name field in it. However, here is a trap. The byte number of this name field is no more than 15. So the process name may be cut off if it is too long. Agent will only read the process name in this name field no matter what the name is before it is cut off. Therefore, when you configure the name tag, be sure wo check the status file and get the name there instead of writing a name you think is right.

As for cmdline，the name is collected from `/proc/$pid/status` and the cmdline is from `/proc/$pid/cmdline`. This file saves the command you need when you start the process. For example, if you start a java process whose name is java through `java -c uic.properties`. And all names of java process are java. So we cannot identify the process from the name. Then we have to refer to the file `/proc/$pid/cmdline`.

The content of Cmdline is your starting command. Precisely， it is your starting command in which the space is replaced with`\0`. It does not matter. Just select and copy it. Please do not add spaces by yourself because no spaces are allowed in the tag of monitor strategy.

`java -c uic.properties` in the example above becomes `java-cuic.properties` in cmdline. There is no need to copy and configure the whole cmdline in the strategy. Although the name tag is fully matched, which means `==` matches name, but cmdline is not. We only need to copy part of the string in cmdline that can separate it from other perocesses. Like:

```
proc.num/cmdline=uic.properties if all(#2) == 0 then alarm()
```

The example above is acceptable. Falcon-Agent does the judging by using `strings.Contains()` after it gets cmdline file.

Does it sounds complex? OK, let me give you an simple explanation. If there are ports monitoring your process, you only have you configure port monitor and skip process monitor. Because when the process stops, the port will not monitor it anymore.


