##Historical data inquiry

Any data pushed in open-falcon can be inquired through API provided by query component afterwards.

##Inquire historical data

Use the port HTTP POST /graph/history to inquire the historical data of certain period in the past. The port cannot be used to inquire the two newly submitted data points. One python example is as follows: 

```
#-*- coding:utf8 -*-

import requests
import time
import json

end = int(time.time()) # starting time
start = end - 3600  # ending time （In example,the data is queried an hour ago）

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
Among which,

1.start: the start time point of the historical data need to be inquired (as the form of UNIX time)

2.end: the end time point of the historical data need to be inquired (as the form of UNIX time)

3.cf: appointed sample mode, can choose from: AVERAGE, MAX, and MIN

4.endpoint_counters: arrays, among which each element is a key value composed of endpoint and counter, among which counter is comprised by metric/sorted(tags), if there is no tags then it is metric by itself.

5.query_api: the monitor address of query module + api

##Inquire newly submitted data

Use the port HTTP POST /graph/last to inquire a newly submitted data point. A bash example is as follows

