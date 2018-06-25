<!-- toc -->

# FAQ about Graph Link

### How to Delete Out-Dated Index?
After monitor data is stopped from reporting, corresponding index will stop updating as well, becoming out-dated index that confuses users. So some users hope that we delete out-dated index.

Our original plan is: Index with data reported is updated once a day through Task module and index that is not updated in the past 7 days will be deleted. However, many users cannot correctly configure the http port of Graph instance, then the index that normally reports monitor data cannot be updated. As a result, the legal index is mistakenly deleted by Task module. 

To solve the issue above, we removed the feature of automatically deleting the out-dated index in the default configuration of Task module. Users are required to trigger deletion of out-dated index that they desire. Here are the steps:

1. Run Task Module, then correctly configure Graph cluster and its http port, which is "index.cluster" in Task configuration. You should not start the deletion of index if the configuration here is not correct, or legal index will be deleted mistakenly.。

2. Run a full update of index data, in other words, run ``` curl -s "$Hostname.Of.Task:$Http.Port/index/updateAll" ```. ```"$Hostname.Of.Task:$Http.Port"``` is the address of http port of Task
PS: Index data are saved in Graph instances. What we do here is to trigger the full update of index in each Graph instances. A more direct method is to run ```curl -s "127.0.0.1:6071/index/updateAll"``` in each Graph instance to trigger Graph instance for full index update (provided the http monitor port of Graph is 6071).

3. Start deleting out-dated index after the full update of index is finished ``` curl -s "$Hostname.Of.Task:$Http.Port/index/delete" ```. Please make sure **the full update of index is finished** before index deletion. A typical schedule is runing a full index update on Saturday and running a index deletion on Sunday, which leaves enough time between index update and index deletion.

We recommend that  **do not delete index unless it is necessary**. If you are determined to delete index, please make sure that index in all Graph instances are fully updated before the index deletion.


### Missing Dashboard Index, Endpoint or Counter not Found?
Those errors may occur when Graph database is manually changed. "Manually changed" includes changing the configuration of database (address, name etc.), deleting or recreating Graph database/datalist, manually editing the content in Graph datalist and so on. When above errors occur, you can solve them through one of the two methods below:

1. Trigger the full index update of Graph to fix the errors that result from manual operation. Just run```curl -s "http://$hostname:$port/index/updateAll"```, in which ```$hostname``` is the address where the server of Graph is，```$port``` is the http monitor port of Graph. Monitor data that have been sent will not be deleted in this way, but it will bring write-read pressure to database in a short time.
2. Delete the data that Graph has already saved and reboot Graph. The data store directory of Graph is ```/home/work/data/6070/``` by default. Monitor data that have been sent will be deleted in this way.



### The curves is Dashboard are null
The display of curves usually has a latency of 3 to 5 report cycles. If no curves are displayed after 5 report cycles, please continue reading.
The graph link in data reporting is `agent -> transfer -> graph -> query -> dashboard`. Users will not see the curves in Dashboard if any link in process does not work. Then, we recommend you to follow the setp below.

1. Make sure all the modules in the Graph link are enabled.
2. Check if there is anything wrong with Dashboard. First, check the log. The default address id ```./var/app.log```. Common reasons of log error include the dependent libraries of Dashboard are not completely installed, local http proxy hijacks the request of visiting and etc. Then, check the configuration of Dashboard. The default configuration files are ```./gunicorn.conf``` and ```./rrd/config.py```. Check the specified visiting address in gunicorn.conf, the query address in config.py, and the configuration of Dashboard database and Graph database.
3. Check Query. First, check the log of Query ```./var/app.log``` to see if there is any error information. Then, check the configuration of Graph list ```./graph_backends.txt``` and the configuration of service ```./cfg.json``` in Query. You can do some quantitive debugging through the script ```./scripts/query```. For example, you report a collection item of "endpoint="ty-op-mon-cloud.us", metric="agent.alive" , you can run ```bash ./scripts/query "ty-op-mon-cloud.us" "agent.alive" ``` to see the data of this collection item. If you can see the data , that means Query, Graph, Transfer, and Agent links work properly.
4. Check Graph. First, check the log of Graph ```./var/app.log``` to see if there is any errors. Then check the configuration of Graph ```./cfg.json``` to see if the right database is configured. Finally, start debugging Graph referring to Chapter ***Graph Debugging***.
5. Check Transfer. First, check the log of Transfer ```./var/app.log``` to see if there are any errors. Then, check the configuration ```./cfg.json``` to see if the feature of sending to Graph cluster is enabled and the cluster list of Graph is correctly configured. Finally, start debugging Transfer refering to Chapter ***Transfer Debugging***.
6. Check Agent. Open the log of Agent Debugging and monitor the status of data reporting.



### Breakpoints in the curves of Dashboard
This problem probably is due to:

1. Malfunction of user's monitor data reporting. Maybe the auto-collection of data is suspended or in irregular cycle.
2. Malfunction of Falcon causes the lost of data monitor. The error probably occurs in the link from Transfer to Graph. Therefore, we can find out the reason through the debugging port of Transfer and Graph.


### High Availability of Graph Data
Graph data is saved in fragment in each Graph instance by default. Once the hard drive of the machine where the the Graph instance is saved breaks down, thr Graph data is lost for good.

To solve this problem, Open-Falcon provides a plan with high availability of Graph data: data multi-printing. Open-Falcon prints the same copy of Graph data onto more than two Graph instances. These identical copies are mirror backups. When one instance does not function normally, other mirror instance backup can substitute for it. Here are steps of how to achieve high availability of Graph data:

1. Configure the node list of Graph in Transfer, allocating more than 2 mirror Graph instances to each node . Transfer will send the same data to mirror Graph instances. For example, you can configure the node of Graph like ``` "g-00" : "host1:6070,host2:6070"```. Graph instances are separated by comma. Therefore, ```host1:6070```  ```host2:6070``` are mirror Graph instances. Pay attention that the version of Transfer cannot below ***0.0.14***, or this feature is not available.
2. Start/reboot Transfer and the data bi-printing or multi-priting. The pressure of Transfer will mount when the number of mirror instances increases. So the number should be in a resonable limit.
3. Configure the Graph list of Query. For now, **one node is only corresponded with one Graph instance in Query**. Take the Transfer in step 1 for example, you can configure the Graph instance of ```g-00``` in Query as ```"g-00":"host1:6070"``` or ```"g-00":"host2:6070"```, but not ```"g-00":"host1:6070,host2:6070"```.
4. When one Graph instance breaks down (eg.hard drive malfunction), change the Graph list in Query, get rid of the malfunctioning Graph instance and substitute mirror instance for it. On on hand, service in upper level is not available during the configuration of Query is under modification, which will only last a couple of minutes. On the other hand, even though one Graph instance is in malfunction, the Graph data on it can be found on other mirror Graph instances, which achieves high availability of data.

Of course, this high availability plan needs more Graph instances, which will multiply the resource Transfer takes. Also, it needs manual failover. So it can be improved in the future.



### How to identify the RRD file that corresponds to the Counter?
When reported to Graph, each Counter is saved as an independent RRD file in the hard drive. Then how to figure out the relation between a Counter and a RRD file? Query module provides a http port for debugging that can help us query.

`info.py`

```python
#!/usr/bin/env python

import requests
import json
import sys

d = [
        {
            "endpoint": sys.argv[1],
            "counter": sys.argv[2],
        },
]
url = "http://127.0.0.1:9966/graph/info"
r = requests.post(url, data=json.dumps(d))
print r.text

```

`Usage`

    python info.py your.hostname your.metric/tag1=tag1-val,tag2=tag2-val



### Graph Debugging
Graph provides multiple http port for debugging, including internal state statistics port, history data query port etc. The script ```./test/debug``` encapsulate some port in form of shell. We will skip the introduction, so please consult those codes on yourself.

**History Data Query Port**
```HTTP:GET, curl -s "http://hostname:port/history/$endpoint/$metric/$tags"``` returns the last 3 pieces of data that Graph received.

```bash
# history data without tags, $endpoint=test.host, $metric=agent.alive
curl -s "http://127.0.0.1:6071/history/test.host/agent.alive"  | python -m json.tool

# history data with tags, $tags='module=graph,pdl=falcon'
curl -s "http://127.0.0.1:6071/history/test.host/qps/module=graph,pdl=falcon"  | python -m json.tool
```

**Internal State Statistics Port**
```HTTP:GET, curl -s "http://hostname:port/statistics/all"``` outputs internal state statistics in json format as follows. These data are collected by Task module and pushed to Falcon for graph display and alarm.

```bash
curl -s "http://127.0.0.1:6071/statistics/all" | python -m json.tool

# output
{
    "data": [
        { // counter of received items 
            "Cnt": 7,						// cnt
            "Name": "GraphRpcRecvCnt",	// name of counter
            "Other": {},					// other infos
            "Qps": 0,						// growth rate of this counter, per second
            "Time": "2015-06-18 12:20:06" // time when this counter takes place
        },
        { // counter of query requests graph received
            "Cnt": 0,
            "Name": "GraphQueryCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-18 12:20:06"
        },
        { // counter of all sent items in query
            "Cnt": 0,
            "Name": "GraphQueryItemCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-18 12:20:06"
        },
        { // counter of info requests graph received
            "Cnt": 0,
            "Name": "GraphInfoCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-18 12:20:06"
        },
        { // counter of last requests graph received
            "Cnt": 3,
            "Name": "GraphLastCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-18 12:20:06"
        },
        { // counter of index updates
            "Cnt": 0,
            "Name": "IndexUpdateAllCnt",
            "Other": {},
            "Time": "2015-06-18 10:58:52"
        }
    ],
    "msg": "success"
}
```


### Transfer Debugging
Transfer provides multiple http port for debugging, including internal state statistics port, transferred data tracing port and etc. The script ```./test/debug``` encapsulate some port in form of shell. We will skip the introduction, so please consult those codes on yourself.


**Transferred Data Tracing Port**
```HTTP:GET, curl -s "http://hostname:port/trace/$endpoint/$metric/$tags"``` outputs internal state statistics in json format. Transfer will filter out data tagged with ```/$endpoint/$metric/$tags```. When you configure filter condition for the first call, it will not return data. Later calls will return the received data up to 3 nodes. This port is used for debugging.

```bash
# trace data without tags, $endpoint=test.host, $metric=agent.alive
curl -s "http://127.0.0.1:8433/trace/test.host/agent.alive"  | python -m json.tool

# trace data with tags, $tags='module=graph,pdl=falcon'
curl -s "http://127.0.0.1:8433/trace/test.host/qps/module=graph,pdl=falcon"  | python -m json.tool
```

**Internal State Statistics Port**
```HTTP:GET, curl -s "http://hostname:port/statistics/all"``` outputs internal state statistics in json format as follows. These data are collected by Task module and pushed to Falcon for graph display and alarm.

```bash
curl -s "http://127.0.0.1:8433/statistics/all" | python -m json.tool

# output
{
    "data": [
        { // counter of items received
            "Cnt": 0,				// counter, total number of items received since transfer started
            "Name": "RecvCnt",	// name of this this counter
            "Other": {},			// other infos
            "Qps": 0,				// growth rate of this counter, items per sec
            "Time": "2015-06-10 07:46:35" // time when this counter takes place
        },
        { // counter of items received from RPC
            "Cnt": 0,
            "Name": "RpcRecvCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items received from HTTP-API
            "Cnt": 0,
            "Name": "HttpRecvCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items received from SOCKET
            "Cnt": 0,
            "Name": "SocketRecvCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items sent to judge
            "Cnt": 0,
            "Name": "SendToJudgeCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items sent to graph
            "Cnt": 0,
            "Name": "SendToGraphCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items sent to graph-migrating
            "Cnt": 0,
            "Name": "SendToGraphMigratingCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of dropped items sent to judge. transfer would drop items if it could not push them to receivers timely
            "Cnt": 0,
            "Name": "SendToJudgeDropCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of dropped items sent to graph
            "Cnt": 0,
            "Name": "SendToGraphDropCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of dropped items sent to graph-migrating
            "Cnt": 0,
            "Name": "SendToGraphMigratingDropCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items transfer failed to send to judge
            "Cnt": 0,
            "Name": "SendToJudgeFailCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items transfer failed to send to graph
            "Cnt": 0,
            "Name": "SendToGraphFailCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of items transfer failed to send to graph-migrating
            "Cnt": 0,
            "Name": "SendToGraphMigratingFailCnt",
            "Other": {},
            "Qps": 0,
            "Time": "2015-06-10 07:46:35"
        },
        { // counter of cached items which would be sent to judge instances
            "Cnt": 0,
            "Name": "JudgeSendCacheCnt",
            "Other": {},
            "Time": "2015-06-10 07:46:33"
        },
        { // counter of cached items which would be sent to graph instances
            "Cnt": 0,
            "Name": "GraphSendCacheCnt",
            "Other": {},
            "Time": "2015-06-10 07:46:33"
        },
        { // counter of cached items which would be sent to grap-migrating instances
            "Cnt": 0,
            "Name": "GraphMigratingCacheCnt",
            "Other": {},
            "Time": "2015-06-10 07:46:33"
        }
    ],
    "msg": "success"
}
```

### Setting the Storage Cycle of Graph Data

Graph module will save the data in the past 5 years by default, and the RRA configuration of data storage is:

```go
// find this in 'graph/rrdtool/rrdtool.go'
func create(filename string, item *model.GraphItem) error {
	now := time.Now()
	start := now.Add(time.Duration(-24) * time.Hour)
	step := uint(item.Step)

	c := rrdlite.NewCreator(filename, start, step)
	c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

	// configure filing policies
	// 1 point per minute for 12 hours
	c.RRA("AVERAGE", 0.5, 1, 720)

	// 1 point per 5 minutes for 2 days
	c.RRA("AVERAGE", 0.5, 5, 576)
	c.RRA("MAX", 0.5, 5, 576)
	c.RRA("MIN", 0.5, 5, 576)

	// 1 point per 20 minutes 一个点存7d
	c.RRA("AVERAGE", 0.5, 20, 504)
	c.RRA("MAX", 0.5, 20, 504)
	c.RRA("MIN", 0.5, 20, 504)

	// 1 point per 3 hours for 3 months
	c.RRA("AVERAGE", 0.5, 180, 766)
	c.RRA("MAX", 0.5, 180, 766)
	c.RRA("MIN", 0.5, 180, 766)

	// 1 point per day for 5 years
	c.RRA("AVERAGE", 0.5, 720, 730)
	c.RRA("MAX", 0.5, 720, 730)
	c.RRA("MIN", 0.5, 720, 730)

	return c.Create(true)
}

```
You can change the storage cycle of Graph data by editing the RRA of rrdtool ``` c.RRA($CF, 0.5, $PN, $PCNT) ```. For the definition of RRA, please refer to the related information of rrdtool.
