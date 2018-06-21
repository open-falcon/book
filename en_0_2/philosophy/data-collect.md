<!-- toc -->

Monitor data is essential in a monitor system for the following analysis, graph, and alarm. How does falcon deal with the problem of data collection?

Let's brainstorm a list of indexes that needs to be collected.

- Common machine loading information: cpu.idle/load.1min/mem.memfree.percent/df.bytes.free.percent...
- Hardware information which system group is interested in: power consumption, fan speed, disk writability...
- Service monitor data: invocation frequency of a port per minute, latency...
- Monitor index of open source software like Database, HBase, Redis, Openstack...

That's quite a lot. The developers of monitor system are not omniscient. They are not capable of programming all the monitor indexes. For example, DBA knows MySQL the best. He knows what index should be collected. The system only needs to provide a port for data pushing and the rest of the work belongs to DBA. If you want to know what the data pushed to Server look like, you can refer to the two pieces of data mentioned in [Design Intention of Tag and HostGroup](tags-and-hostgroup.md). 

Let's focus on the four representative categories above.

**Machine Loading Information**

The collection of this category is universal. We provide an Agent that can be deployed to collect the information. Unlike zabbix that requires configuration on Server for data collection, Falcon does not need configuration. Once the Agent is deployed and the address of Heartbeat and Transfer is correctly configured, the data collection will start automatically, saving user's problem of configuration. For now, Agent only supports 64-bit Linux system not MacOS or Windows.

**Hardware Information**

The scrpit of collecting hardware information is provided by system group. It runs as a plugin relying on Agent. For the introduction of plugin mechanism， please visit [here](plugin.md)。

**Service Monitor Data**

The script of collecting service monitor data is usually synchronized with the code of service. The script becomes online , offline or expanded as the service does. There are numerous java projects in our company. The development teams provided a universal jar pack. Once it is introduced, it will automatically collect the data of invocation frequency of a port, latency and so on. The data are pushed to the monitor system once a minute. Currently, Agent of Falcon provides a simple http port where the data collected by jar pack are sent. Here is an example of data sent to Agent: 

```bash
curl -X POST -d '[{"metric": "qps", "endpoint": "open-falcon-graph01.bj", "timestamp": 1431347802, "step": 60,"value": 9,"counterType": "GAUGE","tags": "project=falcon,module=graph"}]' http://127.0.0.1:1988/v1/push
```

**Index of Open Source Software**

Intense user like DBA usually write their own collection script that is connected to every instance for data collection. Next, they invoke jsonrpc of server for reporting hundreds of thousands pieces of data once a minute. The best plan for reporting the data is sending them in batch of 500 pieces, not all together.
