##User-defined and push data to open-falcon

Not only the data collected by falcon-agent can be pushed in the monitor system, in certain circumstances, some user-defined data indexes can also be pushed in open-falcon, for example:

1.qps of certain online service

2.Online numbers of people of certain operation

3.The respond time of certain port

4.Status code of certain page (500, 200)

5.Failure times of request for certain port

6.Income statistics of certain operation per minute

7.......

##An user-defined and shell script compilled example pushing data to open-falcon
```# Please note that http request body is a json and the json is a list```

```ts=`date +%s`;```
```
curl -X POST -d "[{\"metric\": \"test-metric\", \"endpoint\": \"test-endpoint\", \"timestamp\": $ts,\"step\": 60,\"value\": 1,\"counterType\": \"GAUGE\",\"tags\": \"idc=lg,project=xx\"}]" http://127.0.0.1:1988/v1/push

```

##An example of python and user-defined that push data to open-falcon

```
#!-*- coding:utf8 -*-

import requests
import time
import json

ts = int(time.time())
payload = [
    {
        "endpoint": "test-endpoint",
        "metric": "test-metric",
        "timestamp": ts,
        "step": 60,
        "value": 1,
        "counterType": "GAUGE",
        "tags": "idc=lg,loc=beijing",
    },

    {
        "endpoint": "test-endpoint",
        "metric": "test-metric2",
        "timestamp": ts,
        "step": 60,
        "value": 2,
        "counterType": "GAUGE",
        "tags": "idc=lg,loc=beijing",
    },
]

r = requests.post("http://127.0.0.1:1988/v1/push", data=json.dumps(payload))

print r.text
```

##API detailed instruction

* metric: the core field, indicates the specific measurement of the collecting item, for example is it cpu_idle, memory_free, or qps?
 
* endpoint: indicates the subject (owner) of Metric, for example if the metric is cpu_idle, then Endpoint indicates it is cpu_idle of which machine
* timestamp: indicates the unix time when submitting the data, please note that it is integer and represents seconds
* value: indicates the value of the metric at present time, float64
* step: indicates the reporting period of collecting items for the data, it is quite important for later configure monitor strategy, so it must be specified clearly. 
* counterType: can only choose one from COUNTER or GAUGE. The former indicates the data collecting item is the timer type, and the latter means it is the original value (note the capital and small letters)
oGAUGE: stores the exact values users upload
oCOUNTER: when storing and reflecting, indexes will be calculated as speed, i.e. (current value - previous value) / time interval
* tags: a group of key values divided by commas which further describes and refines metric, it can be an empty tag string, for example idc=lg, service=xbox, etc.. Multiple tags should be divided by commas.

Instruction: The 7 fields should be all specified. 



