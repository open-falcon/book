# 历史数据查询

任何push到open-falcon中的数据，事后都可以通过query组件提供的API，来查询得到。

## 查询历史数据
查询过去一段时间内的历史数据，使用接口 `HTTP POST /graph/history`。该接口不能查询最新上报的两个数据点。一个python例子，如下

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
其中，
1. start: 要查询的历史数据起始时间点（为UNIX时间戳形式）
2. end: 要查询的历史数据结束时间点（为UNIX时间戳形式）
3. cf: 指定的采样方式，可以选择的有：AVERAGE、MAX、MIN
4. endpoint_counters: 数组，其中每个元素为 endpoint和counter组成的键值对, 其中counter是由metric/sorted(tags)构成的，没有tags的话就是metric本身。
5. query_api: query组件的监听地址 + api


## 查询最新上报的数据
查询最新上报的一个数据点，使用接口`HTTP POST /graph/last`。一个bash的例子，如下

```bash
#!/bin/bash
if [ $# != 2 ];then
    printf "format:./last \"endpoint\" \"counter\"\n"
    exit 1
fi

# args
endpoint=$1
counter=$2

# form request body
req="[{\"endpoint\":\"$endpoint\", \"counter\":\"$counter\"}]"

# request 
url="http://127.0.0.1:9966/graph/last"
curl -s -X POST -d "$req" "$url" | python -m json.tool

```
