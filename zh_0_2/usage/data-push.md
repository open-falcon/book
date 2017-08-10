# 自定义push数据到open-falcon

不仅仅是falcon-agent采集的数据可以push到监控系统，一些场景下，我们自定义的一些数据指标，也可以push到open-falcon中，比如：

1. 线上某服务的qps
2. 某业务的在线人数
3. 某个接⼝的响应时间
4. 某个⻚面的状态码(500、200)
5. 某个接⼝的请求出错次数
6. 某个业务的每分钟的收⼊统计
7. ......

## 一个shell脚本编写的，自定义push数据到open-falcon的例子

```
# 注意，http request body是个json，这个json是个列表

ts=`date +%s`;

curl -X POST -d "[{\"metric\": \"test-metric\", \"endpoint\": \"test-endpoint\", \"timestamp\": $ts,\"step\": 60,\"value\": 1,\"counterType\": \"GAUGE\",\"tags\": \"idc=lg,project=xx\"}]" http://127.0.0.1:1988/v1/push

```

## 一个python的、自定义push数据到open-falcon的例子

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

## API详解

- metric: 最核心的字段，代表这个采集项具体度量的是什么, 比如是cpu_idle呢，还是memory_free, 还是qps
- endpoint: 标明Metric的主体(属主)，比如metric是cpu_idle，那么Endpoint就表示这是哪台机器的cpu_idle
- timestamp: 表示汇报该数据时的unix时间戳，注意是整数，代表的是秒
- value: 代表该metric在当前时间点的值，float64
- step: 表示该数据采集项的汇报周期，这对于后续的配置监控策略很重要，必须明确指定。
- counterType: 只能是COUNTER或者GAUGE二选一，前者表示该数据采集项为计时器类型，后者表示其为原值 (注意大小写)
    - GAUGE：即用户上传什么样的值，就原封不动的存储
    - COUNTER：指标在存储和展现的时候，会被计算为speed，即（当前值 - 上次值）/ 时间间隔
- tags: 一组逗号分割的键值对, 对metric进一步描述和细化, 可以是空字符串. 比如idc=lg，比如service=xbox等，多个tag之间用逗号分割

说明：这7个字段都是必须指定
