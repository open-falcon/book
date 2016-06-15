# Introduciton

Monitoring system is the most important part of the DevOps process and even the entire product life cycle. It can give alerts in advance and identify failure, and provide detailed information after events to locate and track down problems. As monitoring system is a full-fledged DevOps product, various open source options are available for it. When a company just starts up, business scale is smaller, and DevOps team is established at initial stage, an open source monitoring system is the most efficient solution that saves time and efforts. With the rapid development of business scale, the number of objects need monitoring are increasing and becoming more complicated. And users of monitoring system will scale from a few of SRE to additional DEVS and SRE. At this point, capacity of monitoring system and users’ service efficiency become the most highlighted problems.


There are many outstanding open source monitoring systems. In the early days, we have been using zabbix. However, with rapid growing businesses and specific requirements of internet companies, the existing open source monitoring systems are not enough to provide the support for performance, scalability and users’ service efficiency.


Given that, in the past year, from the demands of internet companies and user experience and feedback from different various of SRE, SA and DEVS, combined with industry leading internet companies’ monitoring use cases, and from the perspective of monitoring system, we designed and developed Xiaomi’s monitoring system: open-falcon.


**Open-falcon mission aims to be the best open source and easy-to-use enterprise class internet monitoring product.**


##Highlights and features

* Powerful and flexible data collection: automatic discovery, falcon-agent, snmp, user active push and user defined plug-in support, opentsdb data model like (timestamp, endpoint, metric, key-value tags)

* Scale-out capability: support data collocation, alert identification, historical data store and query operations up to gigacycles per second

* High efficient alert policy management: high efficient portal, in support of policy template, template inheritance and overrides, multiple alert modes and callback

* User-friendly alert setting: maximum alert times, later level, alert restore notice, alert pause, different threshold for different period, maintenance period supported

* Efficient graph component: single instance supporting up to 20 million metric of reporting, archiving and storage (one minute per cycle)

* Efficient history data query component: use rrdtool data archiving policy, returning hundreds of one-year metric historical data just in one second

* Dashboard: multi-dimensional data view, user-defined Screen

* High availability: no core single point in overall system, easy operation and maintenance, easy deployment, scale-out capability

* Programming language: entire system’s backend is developed by golang. Portal and dashboard is written by python.


##Architecture

**Note: aggregator component in the dash line is still under development.**

Each server is installed with falcon-agent. Falcon-agent is a daemon program developed by using golang and is designed for various data and metrics in self-discovering gathering single host. These metrics include but not limited to the following areas, totaling more than 200 metrics.

* CPU related
* Disk related
* IO
* Load
* Content related
* Network related
* Port alive, process alive
* ntp offset (plug-in)
* Some process resource consumption (plug-in)
* netstat, ss-related statistical items collection
* Host kernel configuration parameters


Once a host was installed with falcon-agent, it would automatically collect various metrics and actively report, without users implementing configuration in the server (very different form zabbix). One advantage of this is convenient for user’s maintenance and high coverage. Meanwhile, it could also bring server side high pressure. But open-falcon’s single instance at the server side can provide enough performance, and support scale-out capability. Therefore it could automatically collect enough data. Instead it is a good thing for SRE and DEV, which makes it no longer a problem to track down the cause of the accident.


In addition, falcon-agent provides proxy-gateway. User can push data to gateway easily via http port, and gateway can help transfer to server side efficiently.

For information about falcon-agent, please visit our github site: https://github.com/open-falcon/agent


##Data model

Whether the Data Model is powerful and flexible is crucial for service efficiency of monitoring system users. Take zabbix as an example, if reported data is hostname (or ip) and metric, then when users add alert policy and manage alert policy, it can only be implemented within these two dimensions. Let’s take a look at a common scenario:

If hostA disk capacity is less than 5%, then alert will be sent. General servers have two main partitions: root partition and home partition. But in zabbix, two rules need to be added. If a host is hadoop, there are more than 10 disks and over 10 other rules need to be added as well. This is painful, unacceptable, not for automatic (of course zabbix can fix this problem by configuring self-discovering policy. But it is very challenging).

Open-falcon uses the same data format as opentsdb: metric and endpoint plus various groups of key value tags. For example:

```
{
    metric: load.1min,
    endpoint: open-falcon-host,
    tags: srv=falcon,idc=aws-sgp,group=az1,
    value: 1.5,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
{
    metric: net.port.listen,
    endpoint: open-falcon-host,
    tags: port=3306,
    value: 1,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
```

With this data structure, we can configure alerts and dashboard from multi-dimension. Note: endpoint is a special tag.

##Data collection


Transfer can receive data sent from client side, and forward these data to different backend systems for operation after data normalization and inspection. When forwarding data to every backend system, transfer will implement data partition according consistent hash algorithm to support scale-out of backend systems.

Transfer uses two types of port: jsonRpc port and telnet port. Transfer itself is stateless, and when one or multiple devices are not working, there would be no impacts on it. And transfer has high performance, and can forward up to 5 million data per minute.

Transfer currently supports the following business backend: judge, graph, and opentsdb. Judge is a high performance alert identification component developed by our team. Graph is our high performance data storage, archiving, query component. Opentsdb is an open source time series data storage service and can be enabled by transfer configuration file.
In general, transfer has three kinds of data sources:

1.Basic monitored data collected by falcon-agent

2.Data returned from user defined plug-in executed by falcon-agent

3.client library：online business system, using built-in unified perfcounter.jar, actively collecting and reporting data for qps, latency of every PRC port in business systems

Instructions: The three kinds of data above mentioned will all be sent to native proxy-gateway first, and then forwarded to transfer via gateway.

Basic monitoring refers to the monitoring that can be added into any kind of device, such as cpu, mem, net, io, disk and so on. This type of monitoring is fixed, no requiring configuration or providing additional specified parameters by users. As long as the agent runs, it can directly collect and report. Non-basic monitoring is the opposite. For example, if no access to port number is provided, port monitoring won’t work, even I report all of status of 65535 ports. This kind of monitoring needs to be configured by users, and then starts collecting reported monitoring status (including similar configuration trigger monitor in port monitoring, and similar plug-in script monitor in mysql). It is often not referred to as basic monitoring.

##Alerting

Alert identification is executed by judge component. Users configure relevant alert policy and store it in MySQL. Heartbeat server can regularly load the content in MySQL and judge can also communicate with heartbeat server to get relevant alert policy.

Heartbeat sever not just load the content in MySQL, but also can be bound to hostGroup according to template inheritance, template item coverage, alert action coverage, and template, to calculate alert policy finally related to every endpoint and provide relevant calculation results for judge component.

Any data forwarded from transfer to judge can trigger policy-related identification to determine if alert condition is satisfied. If the condition is satisfied, data will be sent to alarm. Then alarm will notify related users by using email, short message or mi messaging tools. It also can execute user pre-configured callback address.

Users can configure alert identification policy in very flexible ways, such as meeting the conditions for n consecutive times, meeting the maximum conditions for n consecutive times, different thread in different time frame, being ignored if in the maintenance period.

Additionally, it can support identification and alert that will rise and fall sharply.

##Query

So far, data have been successfully stored into graph. How can data be read out quickly? For data in the past one hour, one month, or one year, how can it be returned in one second?

This goal can be accomplished by using graph and query component, transfer will forward the data to graph component, and when graph receives the data, it can store the data in the format of rrdtool, and provide it to query RPC port.

Query is enduser-oriented. Once receiving query request, it will go to multiple graphs to query different metric data, summarize the data and return them to users.



##Dashboard

At dashboard homepage, users can search endpoint lists in multi-dimension, namely search relevant endpoint according to reported tags.
##picture

Users can customize multi-metric, and add it to some screen. Therefore, you only need to open the screen to take a look every morning, then you will get the whole picture of service operation.

##picture

You can look up the clear large view. Choose zoom in/out at x-coordinate, quick screening and invert selection. Overall, user’s service efficiency is the top priority.

##picture


##Web portal


An efficient portal can increase user’s service efficiency. This would bring users significant benefits. In such a fast paced society, if we can reduce the burden of SRE and Devs, that couldn't be better.

This is administrative page of the host group and it can be combined with service tree. When host is in and out of the node of service tree, related format will be automatically correlated or relieved.. Therefore, no manual monitoring changes will be need for service online and offline. This can greatly increase efficiency and reduce omissions and false alert.


##picture

Take one simple template as an example. After templates supporting inheritance and policy coverage, template and host group are bound, hosts within the host group will automatically apply all policies of the templates.

##picture

Of course, you can write a simple representation to achieve the goal of monitoring, which is quite convenient for endpoints not having host as a name.

##picture

Adding a representation is also very simple.

##picture



##Storage

For monitoring system, efficient storing and query of historical data is always a challenge.

1.Large volume of data: currently, our monitoring system can report 20 million data per cycle (report cycle contains 1 minute and 5 minutes, each accounting for 50%). During the 24 hours of a day, there never has been low peak of business. Whether day or night, there are always so many data need to be updated.

2.Large quantity of writing operation: For common business system, there is often more readings than writings. It can easily use various caching technologies. Different databases require much more efficient query operation than writing. However, monitoring system is the opposite, requiring more writing than reading. Millions of updating operations per cycle is impossible for common databases (including MySQL, postgresql, mongodb).

3.Efficient query: we said that reading operations of monitoring systems are smaller in quantity, which is relatively smaller compared with writing operation. Monitoring system itself has high requirements for reading operations. Users often need to query hundreds of metrics, including data in the past one day, week, month or year. How can data be returned to users and make mapping? It is a big challenge.

We have made huge amount of investments in open-falcon. According to different using purpose, we divided data into the following two categories: used for mapping and used for data mining.

For data used for mapping, quick query is critical, and data loss is unacceptable. If user wants to query 100 metrics in the past one year, data volume is very large and it is very hard to return data in one second. Even if data can be returned, it is too difficult to render so much data. Sampling also can cause unnecessary consumption and waste. We use the concept of rrdtool as reference. Every time data is stored, sampling and archiving will be automatically executed. Our archiving policies are as follows: historical data retention period is 5 years. To avoid data loss, data archiving will retain 3 copies according to average value sampling, maximum value sampling, and minimum sampling.

```
// One minute a point saving twelve hours
c.RRA("AVERAGE", 0.5, 1, 720)

// five minutes a point saving two days
c.RRA("AVERAGE", 0.5, 5, 576)
c.RRA("MAX", 0.5, 5, 576)
c.RRA("MIN", 0.5, 5, 576)

// twenty minutes a point saving seven days
c.RRA("AVERAGE", 0.5, 20, 504)
c.RRA("MAX", 0.5, 20, 504)
c.RRA("MIN", 0.5, 20, 504)

// three hours a point saving three months
c.RRA("AVERAGE", 0.5, 180, 766)
c.RRA("MAX", 0.5, 180, 766)
c.RRA("MIN", 0.5, 180, 766)

// one day a point  saving one year
c.RRA("AVERAGE", 0.5, 720, 730)
c.RRA("MAX", 0.5, 720, 730)
c.RRA("MIN", 0.5, 720, 730)

```

As for raw data, transfer will forward the copy to hbase, or directly use opentsdb. transfer enables writing data to opentsdb.

##Contributors

* [authors](
* [contributing]()


















