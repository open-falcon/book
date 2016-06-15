# 5.2 About Data Collection

As a monitoring system, it must start with monitoring data, and then it can perform subsequent tasks, such as analyzing, processing, mapping and alerting. How does Falcon deal with the problem about data collection?

First, we have to think about which kind of data we should collect. Imagine that:

* Host load information, the most common information is: cpu.idle/load.1min/mem.memfree.percent/df.bytes.free.percent etc.
* Hardware information, such as power consumption, fan speed, writable or non-writable disks. This is the information that system team would pay attention to
* Service monitoring data, such as the number of call per minute about some port, latency and so on
* nitoring metrics of open source software, including database, HBase, Redis, and Openstack
Facing so many data that are required to collect, the developers of monitoring systems can’t handle all of them. As for MySQL, DBA is the perfect choice. He knows which metrics he should collect. The developers of monitoring system only need to provide a port to push data, and then all teams can take an approach of co-building. Do you want to check out the data that are pushed to Server? You can refer to 2 pieces of json data mentioned in Tag and HostGroup’s Design Concept.

All the above 4 parts are very representative. Let’s explain them one by one.


##Host load information
This is common, so we provide an agent which is deployed on all hosts to collect data. To collect the data you want, you need to configure at server side in zabbix. Unlike zabbix, you don’t need configuring in Falcon. As long as the agent can be deployed on hosts, and heartbeat and Transfer are properly configured, data will be collected automatically. This eliminates problems on user configuration. Currently, the agent only supports 64 bit Linux, not for Mac or Windows.

##Hardware information

Collection script of hardware information is provided by systems team, running as Plugin on agent. For information about Plugin mechanism, please see here.

##Service monitoring data

Collection script of service monitoring metrics usually depends on service’s code. When service is online or scaling, the relevant script is also online or scaling; when service is offline, script is offline too. There are many projects on Java in the company, so research and development team can provide a common jar packet. If you bring in this jar packet, various data will be automatically collected, such as call times of port and latency. Then the collected data will be pushed to monitoring system once per minute. At present, agent of Falcon provides a simple http port. When this jar packet collected data, it would post to native agent. The following is a simple example to push data to agent:
```
curl -X POST -d '[{"metric": "qps", "endpoint": "open-Falcon-graph01.bj", "timestamp": 1431347802, "step": 60,"value": 9,"counterType": "GAUGE","tags": "project=Falcon,module=graph"}]' http://127.0.0.1:1988/v1/push
```
##Monitoring metrics of various open source software

These sections are for large customers. For example, DBA would write some collection scripts on its own, and connect them to MySQL instances for collecting data. Once finished, you can directly call jsonrpc of server side to report data once per minute, even pushing several hundred thousand pieces of data every time. It is a great send mode to make every 500 pieces of data as a batch, instead of sending several hundred thousand pieces of data every time.
