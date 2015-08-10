# 绘图链路常见问题

### 如何清除过期索引
监控数据停止上报后，该数据对应的索引也会停止更新、变为过期索引。过期索引，影响视听，部分用户希望删除之。

我们原来的方案，是: 通过task模块，有数据上报的索引、每天被更新一次，7天未被更新的索引、清除之。但是，很多用户不能正确配置graph实例的http接口，导致正常上报的监控数据的索引 无法被更新；7天后，合法索引被task模块误删除。

为了解决上述问题，我们停掉了task模块自动删除过期索引的功能、转而提供了过期索引删除的接口。用户按需触发索引删除操作，具体步骤为:

1.运行task模块，并正确配置graph集群及其http端口，即task配置文件中index.cluster的内容。此处配置不正确，不应该进行索引删除操作，否则将导致索引数据的误删除。

2.进行一次索引数据的全量更新。方法为 ``` curl -s "$Hostname.Of.Task:$Http.Port/index/updateAll" ```。这里，"$Hostname.Of.Task:$Http.Port"是task的http接口地址。
PS:索引数据存放在graph实例上，这里，只是通过task，触发了各个graph实例的索引全量更新。更直接的办法，是，到每个graph实例上，运行```curl -s "127.0.0.1:6071/index/updateAll"```，直接触发graph实例 进行索引全量更新(这里假设graph的http监听端口为6071)。

3.待索引全量更新完成后，发起过期索引删除 ``` curl -s "$Hostname.Of.Task:$Http.Port/index/delete" ```。运行索引删除前，请务必**确保索引全量更新已完成**。典型的做法为，周六运行一次索引全量更新，周日运行一次索引删除；索引更新和删除之间，留出足够的时间。

在此，建议您: **若无必要，请勿删除索引**；若确定要删除索引，请确保删除索引之前，对所有的graph实例进行一次索引全量更新。


### Dashboard索引缺失、查询不到endpoint或counter
手动更改graph的数据库后，可能会出现上述情况。这里的手动更改，包括：更改graph的数据库配置(数据库地址，名称等)、删除重建graph数据库/表、手动更改graph数据表的内容等。出现上述情况后，可以通过 如下两种途径的任一种 来解决问题，

1. 触发graph的索引全量更新、补救手工操作带来的异常。触发方式为，运行```curl -s "http://$hostname:$port/index/updateAll"```，其中```$hostname```为graph所在的服务器地址，```$port```为graph的http监听端口。这种方式，不会删除已上报的监控数据，但是会对数据库造成短时间的读写压力。
2. 删除graph已存储的数据，并重启graph。默认情况下，graph的数据存储目录为 ```/home/work/data/6070/```。这种方式，会使已上报的数据被删除。



### Dashboard图表曲线为空
图表曲线，一般会有3-5个上报周期的延迟。如果5个周期后仍然没有图表曲线，请往下看。
绘图链路，数据上报的流程为: agent -> transfer -> graph -> query -> dashboard。这个流程中任何一环节出问题，都会导致用户在dashboard中看不到曲线。一个建议的问题排查流程，如下

1. 确保绘图链路的各组件，都已启动。
2. 排查dashboard的问题。首先查看dashboard的日志，默认地址为```./var/app.log```，常见的日志报错原因有dashboard依赖库安装不完整、本地http代理劫持访问请求等。然后查看dashboard的配置，默认为```./gunicorn.conf```和```./rrd/config.py```。确认gunicorn.conf中指定的访问地址，确认config.py中指定的query地址、dashboard数据库配置 和 graph数据库配置。
3. 排查query的问题。首先看下query的日志```./var/app.log```是否有报错信息。然后确认query的graph列表配置```./graph_backends.txt```和 服务配置```./cfg.json```。可以通过脚本```./scripts/query```来做一些定量的debug，比如你上报了一个 endpoint="ty-op-mon-cloud.us", metric="agent.alive" 的采集项，可以运行 ```bash ./scripts/query "ty-op-mon-cloud.us" "agent.alive" ``` 来查看该采集项的数据，如果能够查到数据 则说明query及之前的graph、transfer、agent很可能是正常的。
4. 排查graph的问题。首先看下graph的日志```./var/app.log```是否有报错。然后确认下graph配置文件```./cfg.json```是否配置了正确的数据库db。没有发现问题?启动对graph的debug，方法见***Graph调试***一节。
5. 排查transfer的问题。首先看transfer的日志```./var/app.log```是否有报错。然后确认配置文件```./cfg.json```是否enable了对graph集群的发送功能、是否正确配置了graph集群列表。确认完毕后，仍没有发现问题，怎么办？启动对transfer的debug，方法见***Transfer调试***一节。
6. 排查agent的问题。打开agent的debug日志，观察数据上报情况。



### Dashboard图表曲线有断点
dashboard出现断点，可能的原因为：

1. 用户监控数据上报异常。可能是用户自动的数据采集 被中断，或者上报周期不规律等。
2. falcon系统异常，导致监控数据丢失。问题最可能发生在transfer到graph这个链路上，可以通过transfer&graph的debug接口来确认原因。


### Graph绘图数据高可用
默认情况下，绘图数据以分片的形式、存储在单个graph实例上。一旦graph实例所在的机器磁盘坏掉， 绘图数据就会永久丢失。

为了解决这个问题，Open-Falcon提供了绘图数据高可用的解决方案: 数据双打/多打，即 将同一份绘图数据 同时打到2+个graph实例上、使这2+个graph实例的数据完全相同、互为镜像备份，当发生故障时 就可以用 镜像备份实例 来替换 故障的实例。具体的，绘图数据高可用的实现过程，如下:

1. 配置transfer的graph节点列表，使每个节点都有2+个互为镜像的graph实例；transfer会向 互为镜像的几个graph实例 发送相同的数据。eg.你可以配置graph节点 形如``` "g-00" : "host1:6070,host2:6070"```，多个graph实例之间以逗号隔开，这样 ```host1:6070``` 和 ```host2:6070```就会互为镜像。注意，transfer版本应不低于***0.0.14***，否则 不能支持此功能。
2. 启动/重启transfer，开始数据双打或者多打。transfer的压力，会随着镜像数量的增加而增大，所以 需要合理控制graph镜像的数量。
3. 配置query的graph列表。当前，query只支持 **一个节点对应一个graph实例**。以第1步的transfer配置为例，你可以配置query的graph节点```g-00```的graph实例为 ```"g-00":"host1:6070"``` 或者 ```"g-00":"host2:6070"```、而不能是 ```"g-00":"host1:6070,host2:6070"```
4. 当某个graph实例发生故障(eg.磁盘故障)时，修改query的graph列表、踢掉坏死的graph实例、并用其镜像实例顶替之。一方面，更上层的业务只会在query配置修改期间不可用，这个过程可以控制在分钟级别。另一方面，即使坏掉了一个graph实例，其上绘图数据 也可以从镜像实例上找到，从而实现了数据的高可用。

当然，这个高可用方案 需要增加更多的graph实例、会使transfer资源消耗倍增、需要手动进行故障切换，还有很多的改进空间。



### 如何确定某个counter对应的rrd文件
每个counter，上报到graph之后，都是以一个独立的rrd文件，存储在磁盘上。那么如何确定counter和rrd的对应关系呢？
query组件提供了一个调试用的http接口，可以帮助我们查询到该对应关系

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

`使用方法`

    python info.py host1 cpu.idle



### Graph调试
graph以http的方式提供了多个调试接口。主要有 内部状态统计接口、历史数据查询接口等。脚本```./test/debug```将一些接口封装成了shell的形式，可自行查阅代码、不在此做介绍。

**历史数据查询接口**```HTTP:GET, curl -s "http://hostname:port/history/$endpoint/$metric/$tags"```，返回graph接收到的、最新的3个数据。

```bash
# history没有tags的数据,$endpoint=test.host, $metric=agent.alive
curl -s "http://127.0.0.1:6071/history/test.host/agent.alive"  | python -m json.tool

# history有tags的数据,$tags='module=graph,pdl=falcon'
curl -s "http://127.0.0.1:6071/history/test.host/qps/module=graph,pdl=falcon"  | python -m json.tool
```

**内部状态统计接口**```HTTP:GET, curl -s "http://hostname:port/statistics/all"```，输出json格式的内部状态数据，格式如下。这些内部状态数据，被task组件采集后push到falcon系统，用于绘图展示、报警等。

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


### Transfer调试
transfer以http的方式提供了多个调试接口。主要有 内部状态统计接口、转发数据追踪接口等。脚本```./test/debug```将一些http接口封装成了shell的形式，可自行查阅代码、不在此做介绍。

**转发数据追踪接口** ```HTTP:GET, curl -s "http://hostname:port/trace/$endpoint/$metric/$tags"```，输出json格式的trace数据。transfer会过滤出以```/$endpoint/$metric/$tags```标记的数据，第一次调用时设置过滤条件、不会返回数据，之后每次调用返回已接收到的数据、最多3个点。这个接口主要用于debug。

```bash
# trace没有tags的数据,$endpoint=test.host, $metric=agent.alive
curl -s "http://127.0.0.1:8433/trace/test.host/agent.alive"  | python -m json.tool

# trace有tags的数据,$tags='module=graph,pdl=falcon'
curl -s "http://127.0.0.1:8433/trace/test.host/qps/module=graph,pdl=falcon"  | python -m json.tool
```

**内部状态统计接口**```HTTP:GET, curl -s "http://hostname:port/statistics/all"```，输出json格式的内部状态数据，格式如下。这些内部状态数据，被task组件采集后push到falcon系统，用于绘图展示、报警等。

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

### 设置绘图数据的存储周期
Graph组件默认保存5年的数据，存储数据的RRA配置为:

```go
// find this in 'graph/rrdtool/rrdtool.go'
func create(filename string, item *model.GraphItem) error {
	now := time.Now()
	start := now.Add(time.Duration(-24) * time.Hour)
	step := uint(item.Step)

	c := rrdlite.NewCreator(filename, start, step)
	c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

	// 设置各种归档策略
	// 1分钟一个点存 12小时
	c.RRA("AVERAGE", 0.5, 1, 720)

	// 5m一个点存2d
	c.RRA("AVERAGE", 0.5, 5, 576)
	c.RRA("MAX", 0.5, 5, 576)
	c.RRA("MIN", 0.5, 5, 576)

	// 20m一个点存7d
	c.RRA("AVERAGE", 0.5, 20, 504)
	c.RRA("MAX", 0.5, 20, 504)
	c.RRA("MIN", 0.5, 20, 504)

	// 3小时一个点存3个月
	c.RRA("AVERAGE", 0.5, 180, 766)
	c.RRA("MAX", 0.5, 180, 766)
	c.RRA("MIN", 0.5, 180, 766)

	// 1天一个点存5year
	c.RRA("AVERAGE", 0.5, 720, 730)
	c.RRA("MAX", 0.5, 720, 730)
	c.RRA("MIN", 0.5, 720, 730)

	return c.Create(true)
}

```
你可以通过修改rrdtool的RRA ``` c.RRA($CF, 0.5, $PN, $PCNT) ```，来设置Graph的数据存储周期。关于RRA的概念，请查阅rrdtool的相关资料。
