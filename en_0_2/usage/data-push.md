<!-- toc -->

# Customization of Pushing Data to Open-Falcon

Not only the data collected by Falcon-Agent can be pushed to the monitor system, so can some customized data index in some situation. For example：

1. Qps of a online service
2. Online user number of a service
3. Response time of a port
4. State code of a page
5. Failure counter of a port's request 
6. Income of a service per minute
7. ......

## Example of Customization of Pushing Data to Open-Falcon Written in Shell Script

```
# Notice: the body of http request is a json  and this json is a list

ts=`date +%s`;

curl -X POST -d "[{\"metric\": \"test-metric\", \"endpoint\": \"test-endpoint\", \"timestamp\": $ts,\"step\": 60,\"value\": 1,\"counterType\": \"GAUGE\",\"tags\": \"idc=lg,project=xx\"}]" http://127.0.0.1:1988/v1/push

```

## Example of Customization of Pushing Data to Open-Falcon Written in Python

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

## Details about API

- Metric: the most important field. It defines what this index measures. Is it cpu_idle, memory_free, or qps?
- Endpoint: marking the owner of a Metric. If the metric is cpu.idle, then the Endpoint shows which machine this cpu_idle belongs to.
- Timestamp: the unix timestamp when the data is pushed, which is an integer means seconds.
- Value: the value of metric at that time point, float64.
- Step: marking the reporting cycle of this data index. It is very important to the following configuration of monitor stragegy, which must be speficied.
- CounterType: can only be COUNTER or GAUGE (all in capital letters). The former one means it is a type of timer, and the latter one shows the original value.
    - GAUGE：saving the value as it is uploaded by user
    - COUNTER：calculated as speed during storage and display, which equals (current value - previous value) / time interval
- Tags: a group of values separated by comma. They are the further and detailed description of the metric. It can be empty string or like idc=lg or service=xbox等

PS：These seven fields must be specified.
