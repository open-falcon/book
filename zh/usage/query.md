# 历史数据查询

任何push到open-falcon中的数据，事后都可以通过query组件提供的API，来查询得到。

## 一个python的例子

```python
#-*- coding:utf8 -*-

import requests
import time
import json

end = int(time.time()) # 起始时间戳
start = end - 3600  # 截至时间戳 （例子中为查询过去一个小时的数据）

d = {
        "start": start,
        "end": end,
        "cf": "AVERAGE",
        "endpoint_counters": [
            {
                "endpoint": "host1",
                "counter": "cpu.idle",
            },
            {
                "endpoint": "host1",
                "counter": "load.1min",
            },
        ],
}

query_api = "http://127.0.0.1:9966/graph/history"
r = requests.post(query_api, data=json.dumps(d))
print r.text

```

## API详解

1. start: 要查询的历史数据起始时间点（为UNIX时间戳形式）
1. end: 要查询的历史数据结束时间点（为UNIX时间戳形式）
1. cf: 指定的采样方式，可以选择的有：AVERAGE、MAX、MIN
1. endpoint_counters: 数组，其中每个元素为 endpoint和counter组成的键值对, 其中counter是由metric/sorted(tags)构成的，没有tags的话就是metric本身。
1. query_api: query组件的监听地址 + api


