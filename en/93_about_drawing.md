# 9.3 About drawing

##Problems about drawing link

###How to eliminate the outdated index

When the monitoring data stopped reporting, the corresponding index to the data will stop updating and become outdated. Outdated index will have impact on seeing and hearing, so some users want to remove it.

Our original plan was: by task module, the indexes the data reported is updated once a day, and those that was not updated for 7 days will be eliminated. However, many users cannot correctly deploy the http interface of graph instance, leading the indexes of the normally reported monitoring data to fail to be updated. After 7 days, the legitimate indexes are deleted by mistake by task module.

In order to solve these problems, we stopped the function that the task module can automatically delete expired indexes, and provided an interface to delete the expired indexes. Users trigger the index deleting operations by demand, and the concrete steps are as follows:

1. Run the task module, and correctly deploy graph cluster and its http port, namely the content in index.cluster in task deploy document. We shouldn’t delete any index if it is not correctly deployed here, or it will lead to the result that the index data are deleted by mistake. 

2. Update all the index data. The method is: ```curl -s "$Hostname.Of.Task:$Http.Port/index/updateAll"```.
Here, "$Hostname.Of.Task:$Http.Port" is the http address of task. PS: index data is stored in graph instance. Here, all the index updating of every graph instance is trigged by task. The more straightforward method is to run ```curl -s "127.0.0.1:6071/index/updateAll"``` on every graph instance to directly trigger the graph instance to update all the data (here, assuming that the http monitoring port of graph is 6071).

3. After updating all the index data, start the expired index elimination ```curl -s "$Hostname.Of.Task:$Http.Port/index/delete"```.Before you delete the index, make sure that the total amount of the index update is complete. Typical method is to update all the indexes every Saturday and elimination index every Sunday. Leave enough time between index updating and deleting.
4. 
Here, we recommend that: if it is not necessary, do not delete the index; if you are sure to delete the index, be sure to update all the indexes of all the graph instances before deleting.

###Dashboard index deficiency causes endpoint or counter to fail to be queried

The case above may occur after you manually change the graph database. Here, the manual changes include: changing the graph database configuration (database address, name, etc.), removing and rebuilding the graph database/ table, manually changing the contents of the graph data table and so on. After one of the cases above happens, you can solve the problem by either of the following two ways:

1.Trigger the full index updating of graph to remedy the error caused by manual operating. The method is to run ```curl -s "http://$hostname:$port/index/updateAll"```, therein, ```$hostname``` is the graph server address, ```$port```为is the http monitoring port of graph.In this way, the reported data will not be deleted but it may bring reading and writing pressure to the database temporarily. 

2.Delete the data graph stored and restart graph. By default, the data storing directory of graph is```/home/work/data/6070/```. In this way, the reported data will be deleted

###Dashboard graph curve is null

Graph curve generally has 3-5 reporting period of delay. If there is still not a graph curve after five cycles, please read on. Drawing links, data reporting process: agent -> transfer -> graph -> query -> dashboard. Any problem during the process will cause the users to fail to see the curve in the dashboard. One troubleshooting process suggested is as follows:

1.Make sure that all elements of drawing link have started.

2.Troubleshoot the problem of dashboard. Firstly, check the dashboard log, default address ```./var/app.log```. The common reasons of log reporting error are the installing of dashboard dependent library is incomplete, local http agent keeps from access query and so on. Then check the configuration of dashboard, default ```./gunicorn.conf``` and ```./rrd/config.py```. Verify the access address specified in gunicorn.conf. Verify the query address specified in config.py, dashboard database configuration and graph database configuration. 

3.Troubleshoot the problems of query. Firstly, check if there is error information in query log ```./var/app.log```. Then make sure the graph list configuration of query ```./graph_backends.txt``` and service configuration ```./cfg.json```. You can make some quantitative debug by script ```./scripts/query```, for example, when you report a collection item  endpoint="ty-op-mon-cloud.us", metric="agent.alive" , you can run ```bash ./scripts/query "ty-op-mon-cloud.us" "agent.alive"``` to check the data of it. If you can find the data, it means that the query and previous graph, transfer, agent are very likely to be normal. 

4.Troubleshoot the problems of graph. Firstly check if there is error in graph log ```./var/app.log``` Then check whether the graph configuration file ```./cfg.json``` has deployed correct database db. No problem? Start the debug to graph, referring to section Graph bug.

5.Troubleshoot the problems of transfer. Firstly check if there is error in transfer log ```./var/app.log``` Then check whether the sending function to graph cluster is enabled by configuration file ```./cfg.json``` and whether graph cluster list is correctly deployed. If there is still no problem after verifying, what should be done? Start the debug to transfer, referring to section Transfer debug. 

6.Troubleshoot the problems of agent. Open the debug log of agent to observe the data reporting condition. 

###Dashboard graph curve has breakpoints

Dashboard has breakpoints, the reasons may be:

1.User monitoring data reporting errors. It is probably because Users’ automatic data collection is interrupted or reporting period is irregular and so on.

2.Falcon system errors, resulting in loss of monitoring data. Problems are most likely to occur in the link from transfer to graph and we can confirm the cause by the debug interface of transfer & graph. 

###Graph drawing data high availability

By default, the drawing data is stored on a single graph instance in fragmented form. Once the machine disk where there are graph instances goes bad, the drawing data will be permanently lost.

To solve this problem, Open-Falcon provides drawing data highly available solution: data double /multiple print, that is, to print the  same drawing data to more than 2 graph instances so as to make the data of the two graph instances the same and become the mirror backup of each other. When a failure occurs, you can use the mirror backup instance to replace the failed instance. Specifically, the process of drawing data to achieve high availability is as follows:

1.Deploy  the graph node list of transfer to make every node has more than two graph instances which are the mirror backup of each other, and transfer will send the same data to these graph instances. eg. You can deploy graph node like ```"g-00" : "host1:6070,host2:6070"```, multiple graph instances are separated with a comma, thus  ```host1:6070``` and ```host2:6070``` are the mirror backup of each other. Notice, the version of transfer shouldn’t be lower than 0.0.14, or this function will not be supported.

2.Start/restart transfer to start data double or multiple printing. The pressure of transfer will increase with the increase of mirror quantity, so the amount of graph mirror needs to be controlled reasonably. 

3.Deploy the graph list of query. At present, query only supports one node corresponded to one instance. Take the transfer configuration in Step 1 for example. You may deploy the graph instance of query graph node g-00 as  ```"g-00"```:"host1:6070"``` or ```"g-00":"host2:6070"``` instead of ```"g-00":"host1:6070,host2:6070"```

4.When one graph instance breaks down (eg. Disk trouble), modify the graph list of query, eliminate the dead graph instance and replace it with its mirror backup. On the one hand, the upper business is not available only during the period of query configuration modification, and this process can be controlled to the minute level. On the other hand, even if one graph instance is down, its drawing data can still be found on its mirror backup instance so as to complete the high availability of data.

Of course, the high availability scheme needs to add more graph instances so that the transfer resource consumption redoubles and needs manual disturbance switching. There is still a long way to go to improve.

###How to confirm the rrd file corresponding to one counter

Each counter is stored on the disk as a separate rrd file after reported to the graph. Then how to determine the correspondence between the counter and rrd? Query component provides a http interface for debugging, and it can help us to query this correspondence.
```
info.py
#!/usr/bin/env python
import requestsimport jsonimport sys

d = [
        {
            "endpoint": sys.argv[1],
            "counter": sys.argv[2],
        },
]
url = "http://127.0.0.1:9966/graph/info"
r = requests.post(url, data=json.dumps(d))print r.text
```
Using method:
```
python info.py your.hostname your.metric/tag1=tag1-val,tag2=tag2-val
```

###Graph debugging

Graph provides a plurality of debugging interfaces by way of http. There mainly are internal state statistics interface, historical data query interface and so on. The script ```./test/debug``` encapsulates some interfaces by the form of shell. You can look for the code on your own, but not in this presentation.

**Historical data query interface HTTP:GET**, ```curl -s "http://hostname:port/history/$endpoint/$metric/$tags"```, return the latest 3 data graph received.
```
# history without tagsdata,$endpoint=test.host, $metric=agent.alive
curl -s "http://127.0.0.1:6071/history/test.host/agent.alive"  | python -m json.tool
# history with tagsdata,$tags='module=graph,pdl=falcon'
curl -s "http://127.0.0.1:6071/history/test.host/qps/module=graph,pdl=falcon"  | python -m json.tool
```
**Internal state statistics interface HTTP:GET**, ```curl -s "http://hostname:port/statistics/all"```, output the internal state data of json format, as follows. These data are collected by task component and pushed to the falcon system to show drawing and make alert.
```
curl -s "http://127.0.0.1:6071/statistics/all" | python -m json.tool
# output
{
    "data": [
        { // counter of received items 
            "Cnt": 7,                        // cnt
            "Name": "GraphRpcRecvCnt",    // name of counter
            "Other": {},                    // other infos
            "Qps": 0,                        // growth rate of this counter, per second
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

###Transfer debugging

Transfer provides a plurality of debugging interfaces by way of http. There mainly are internal state statistics interface, historical data query interface and so on. The script ```./test/debug``` encapsulates some interfaces by the form of shell. You can look for the code on your own, but not in this presentation.

**Transmitting data trace interface** ```HTTP:GET, curl -s "http://hostname:port/trace/$endpoint/$metric/$tags"```, output data by json format. Transfer can filter the data marked by ```/$endpoint/$metric/$tags```. You should set the filtration condition at the first calling and it doesn’t return any data, then it returns the received data once calling, at most 3 points. This interface is mainly used to debug.
```
# trace without tags data,$endpoint=test.host, $metric=agent.alive
curl -s "http://127.0.0.1:8433/trace/test.host/agent.alive"  | python -m json.tool
# trace with tags data,$tags='module=graph,pdl=falcon'
curl -s "http://127.0.0.1:8433/trace/test.host/qps/module=graph,pdl=falcon"  | python -m json.tool
```
**Internal state statistics interface** ```HTTP:GET, curl -s "http://hostname:port/statistics/all"```, output the internal state data of json format, as follows. These data are collected by task component and pushed to the falcon system to show drawing and make alert. 
```
curl -s "http://127.0.0.1:8433/statistics/all" | python -m json.tool
# output
{
    "data": [
        { // counter of items received
            "Cnt": 0,                // counter, total number of items received since transfer started
            "Name": "RecvCnt",    // name of this this counter
            "Other": {},            // other infos
            "Qps": 0,                // growth rate of this counter, items per sec
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

###Set the storage cycle of drawing data

Graph component stores the data of 5 years by default and the RRA configuration of them are:
```
// find this in 'graph/rrdtool/rrdtool.go'func create(filename string, item *model.GraphItem) error {
    now := time.Now()
    start := now.Add(time.Duration(-24) * time.Hour)
    step := uint(item.Step)

    c := rrdlite.NewCreator(filename, start, step)
    c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

    // Setting all kinds of filing strategies
    // 1 minute one point stored 12h
    c.RRA("AVERAGE", 0.5, 1, 720)

    // 5m one point stored 2d
    c.RRA("AVERAGE", 0.5, 5, 576)
    c.RRA("MAX", 0.5, 5, 576)
    c.RRA("MIN", 0.5, 5, 576)

    // 20m one point stored 7d
    c.RRA("AVERAGE", 0.5, 20, 504)
    c.RRA("MAX", 0.5, 20, 504)
    c.RRA("MIN", 0.5, 20, 504)

    // 3h one point stored 3months
    c.RRA("AVERAGE", 0.5, 180, 766)
    c.RRA("MAX", 0.5, 180, 766)
    c.RRA("MIN", 0.5, 180, 766)

    // 1 day one point stored 5year
    c.RRA("AVERAGE", 0.5, 720, 730)
    c.RRA("MAX", 0.5, 720, 730)
    c.RRA("MIN", 0.5, 720, 730)

    return c.Create(true)
}
```
You can modify the RRA of rrdtool ```c.RRA($CF, 0.5, $PN, $PCNT)``` to set the data storage cycle of graph. For the concept of RRA, please refer to rrdtool.