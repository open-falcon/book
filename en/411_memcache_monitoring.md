##Memcache monitoring

We have introduced the usual monitoring data source in section of Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The data of Memcache can be collected by collecting script memcached-monitor.

##Operating principle

Memcached-monitor is a cron, and the collecting script is run per minute.
```Memcached-monitor```.py can automatically detect the port of Memcached, and connect to Memcached instance to collect some monitoring index, for example get_hit_ratio, usage and so on, then assembling to the normative format of open-falcon to post to the host falcon-agent. Falcon-agent provides a http interface, and as for the using method, please refer to the instances in Data Collection.

For example, we have 1000 machines deployed Memcached instance, and we can deploy 1000 crons for the 1000 machines, i.e. it is one-to-one corresponded to the Memcached instance.

Notice, script ```memcached-monitor.py``` automatically finds the Memcached port by ````ps -ef |grep memcached|grep -v grep |sed -n 's/.* *-p *\([0-9]\{1,5\}\).*/\1/p```. If port is not designated by –p when Memcached initiates, the self-finding would be failed, and at this time, the script needs to be modified and designate the very port.
