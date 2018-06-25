<!-- toc -->

It is a complicated topic, so please be patient while reading.

Let's start with data pushing. The monitor system deployed Agent in every machine for collecting loading information, like cpu、memory、disk、io and network. But for business monitor data, like the cps and latency of invoking a port, they are not able to be collected by Agent. They need to be pushed by the business side. And so do the monitor data related to MySQL and Redis. 

Therefore, the server end of the monitor system requires a port specification and everyone needs to invoke this port user to push the data. Take cpu.idle that Agent reports for example:

```
{
    "metric": "cpu.idle",
    "endpoint": `hostname`,
    "tags": "",
    "value": 12.34,
    "timestamp": `date +%s`,
    "step": 60,
    "counterType": "GAUGE"
}
```

Metric is the name of monitor index, like disk.io.util, load.1min, df.bytes.free.percent. Endpoint is the entity that the index is attached to. For example, cpu.idle apparently is attached to the its machine, so endpoint is the name of the machine for the machine loading information collected by Agent. I will skip tags for now. Value is the value of the index. Timestamp is measured in second. Step is a concept in RRDtool. Falcon uses RRDtool for data storage and step is for telling RRDtool how often the data are reported. CounterType is also a concept in RRDtool, which currently supports GAUGE and COUNTER two types.

Tags is important. So I give another example for an easier understanding. Like the latency of invoaking a method:

```
{
    "metric": "latency",
    "endpoint": "11.11.11.11:8080",
    "tags": "project=falcon,module=judge,method=QueryHistory",
    "value": 12.34,
    "timestamp": `date +%s`,
    "step": 60,
    "counterType": "GAUGE"
}
```

Just like tags in a blog, tags can tag the pushed data that can facilitate filing. The three tags in the example above means: the latency is about the latency of invoking QueryHistory method; the module is Judge; the project is falcon.

It is much easier for us to configure the monitor of these data. For example, If we want the alarm to be triggerred when the latency of invoking all the methods in falcon-judge is over 0.5, then we can configure this expression (in portal): 

```
each(metric=latency project=falcon module=judge)
```

And the threshold condition is all(#1)>0.5.

We do not speficy the method, so it is valid to all the methods. This is the power of tag that one configuration is all you need.


----------


Users can add tags for data pushed by business side. But what if they are collected by Agent? What tags should be added to data like cpu.idle and load.1min? There are no tags for them. So here comes the question of configure the alarm since we cannot give each machine a configuration.

```
each(metric=latency endpoint=host1)
```

The company has 10,000 machines. This is a nightmare.

Tag is actually a way of categorization. If data cannot claim its category when they are being pushed, then we have to **categorize the data in the upper level mannually**. And that is what HostGroup is for.

Let's assume that we put all the machines that the falcon project uses in a HostGroup (named sa.dev.falcon that shows the department information). We bind a policy template to it and the policy is valid to all the machines in the HostGroup. When users add machines to sa.dev.falcon for expansion, the policy automatically becomes valid to the new machines. Users can just delete the machine from the HostGroup when they want it to be offline. 

To sum up, HostGroup is a tool for categorizing the endpoint.

Imagine a piece of data has been pushed. How can we know whether it should trigger the alarm or not? What we need to do is to find the expression and strategy that are associated to the data. It is easy to find the expression. Just check whether the tag in the expression is the subset of the pushed data or not. What about strategy? The data includes endpoint. We can check whether the endpoint belongs to certain HostGroups. Since the hostgroup is bound to template, we can find the strategy in the template.


----------



But the actual operation can be a little difficult because HostGroup is a flat structure. Here is an example.

- We put all the machines of our company in a hostgroup and configure a disk monitor. When is in malfunction, the system sends the alarm to system group.
- User A and B manage three projects (including falcon) together. They put the machines of the three projects in a group and configure a machine loading monitor of cpu.idle and df.bytes.free.percent called sa.dev.common for now. And the alarm is sent to user A and B.
- The io pressure of Falcon's disk is usually higher than the io of the other two projects, which is totally normal. Like we configure the alarm threshold of disk.io.util is 40, but usually 80 is the threshold of falcon. Therefore, we need to create a new template that inherit from sa.dev.common and bind to falcon to overwrite the configuration of parent template that gives a new alarm threshold.
- Falcon has a judge module that monitor port 6080. We need to put all the machines of Judge in a HostGroup and configure its port monitor.

Here is the problem. If five more machines are added to Judge, these five machines need to be added in every group of the four.

It is kind of inconvenient. So in our company, we actually use falcon with the machine management system. The system manages the machines following the structure of a tree. A machine is added to the subnode and the upper node has the machine too.

Therefore, if you use falcon in a smaller scale, you are free to use HostGroup. If it is a bigger scale, then it demands a further development that combines falcon to inner machine management system. If you do not have a machine management system, please wait for us to provide one in the near future.
