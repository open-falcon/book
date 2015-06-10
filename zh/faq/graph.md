# 绘图链路常见问题
### 在dashboard中，索引缺失、查询不到endpoint或counter
手动更改graph的数据库后，可能会出现上述情况。这里的手动更改，包括：更改graph的数据库配置(数据库地址，名称等)、删除重建graph数据库/表、手动更改graph数据表的内容等。出现上述情况后，可以通过 如下两种途径的任一种 来解决问题，

1. 删除graph已存储的数据，并重启graph。默认情况下，graph的数据存储目录为 ```/home/work/data/6070/```。这种方式，会使已上报的数据被删除。
2. 触发graph的索引全量更新、补救用户手工操作带来的异常。触发方式为，运行```curl -s "http://$hostname:$port/index/updateAll"```，其中```$hostname```为graph所在的服务器地址，```$port```为graph的http监听端口。这种方式，不会删除已上报的监控数据，但是会对数据库造成短时间的读写压力。

### 在dashboard中，图表曲线为空
图表曲线，一般会有3-5个上报周期的延迟。如果5个周期后仍然没有图表曲线，请往下看。
绘图链路，数据上报的流程为: agent -> transfer -> graph -> query -> dashboard。这个流程中任何一环节出问题，都会导致用户在dashboard中看不到曲线。一个建议的问题排查流程，如下

1. 确保绘图链路的各组件，都处于启动状态。
2. 排查dashboard的问题。首先查看dashboard的日志，默认地址为```./var/app.log```，常见的日志报错原因有dashboard依赖库安装不完整、本地http代理劫持访问请求等。然后查看dashboard的配置，默认为```./gunicorn.conf```和```./rrd/config.py```。确认gunicorn.conf中指定的访问地址，确认config.py中指定的query地址、dashboard数据库配置 和 graph数据库配置。
3. 排查query的问题。首先看下query的日志```./var/app.log```是否有报错信息。然后确认query的graph列表配置```./graph_backends.txt```和 服务配置```./cfg.json```。可以通过脚本```./scripts/query```来做一些定量的debug，比如你上报了一个 endpoint="ty-op-mon-cloud.us", metric="agent.alive" 的采集项，可以运行 ```bash ./scripts/query "ty-op-mon-cloud.us" "agent.alive" ``` 来查看该采集项的数据，如果能够查到数据 则说明query及之前的组件graph、transfer、agent很可能是正常的。
4. 排查graph的问题。首先看下graph的日志```./var/app.log```是否有报错。然后确认下graph配置文件```./cfg.json```是否配置了正确的数据库db。没有发现问题？好，不必在graph上过多停留、去排查transfer的问题。
5. 排查transfer的问题。首先看transfer的日志```./var/app.log```是否有报错。然后确认配置文件```./cfg.json```是否enable了对graph集群的发送功能、是否正确配置了graph集群列表。确认完毕后，仍没有发现问题，怎么办？启动对transfer的debug，具体见***transfer调试***一节。
6. 排查agent的问题。打开agent的debug日志，观察数据上报情况。

### 在dashboard中，图表曲线有断点
dashboard出现断点，可能的原因为：

1. 用户监控数据上报异常。可能是用户自动的数据采集 被中断，或者上报周期不规律等。
2. falcon系统异常，导致监控数据丢失。问题最可能发生在transfer到graph这个链路上，可以通过transfer的debug接口来确认原因。

### transfer调试
transfer以http的方式，提供了多个调试接口。主要有 内部状态统计接口、转发数据追踪接口等。脚本```./test/debug```将这些http接口封装成了shell的形式。graph组件提供了类似的调试接口，不再赘述。

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

**转发数据追踪接口** ```HTTP:GET, curl -s "http://hostname:port/trace/$endpoint/$metric/$tags"```，输出json格式的trace数据。transfer会追踪 以```/$endpoint/$metric/$tags```标记 的数据，并将最新的3个输出。这个接口主要用于debug。

```bash
curl -s "http://127.0.0.1:8433/trace/test.endpoint.niean.1/test.metric.niean.1/tag0=tag0-niean-1,tag1=tag1-niean-1,tag2=tag2-niean-1"  | python -m json.tool

# output
{
    "data": [
        {
            "Value": { // the first traced item
                "counterType": "GAUGE",
                "endpoint": "test.endpoint.niean.1",
                "metric": "test.metric.niean.1",
                "step": 60,
                "tags": {
                    "tag0": "tag0-niean-1",
                    "tag1": "tag1-niean-1",
                    "tag2": "tag2-niean-1"
                },
                "timestamp": 1433924465,
                "value": 9
            }
        }
    ],
    "msg": "success"
}

```
