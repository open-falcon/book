##monitoring windows platform

We have introduced the usual monitoring data source in section of Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The running index of switch collection: we can write a python to collect each item of running index of switch by SNMP protocol, including memory usage, CPU usage, disk usage, network traffic, etc.

We can collect the monitoring index of windows host directly by windows_collect script.

###Usage
* Modify the configuration parameter at the head of the script according to the real deployment condition. 
* Change the mysql into utf8 in graph to support Chinese. This step is important because windows mac may be Chinese.
* Test: python windows_collect.py
* Run windows plan mission and complete

###The environment tested:
* windows 10
* windows 7
* windows server 2012
________________________________________
Otherwise you can use the golang version windows agent: https://github.com/LeonZYang/agent

