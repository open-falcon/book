# 数据采集相关问题

#### 如何验证数据是否在上报

数据链路是：`agent->transfer->judge and graph`，judge有一个http接口可以验证`agent->transfer->judge`这条链路，比如judge的http端口是6081，可以这么访问验证：

```bash
curl http://127.0.0.1:6081/history/$endpoint/$counter

# $endpoint和$counter是变量，举个例子：
curl http://127.0.0.1:6081/history/host01/cpu.idle

# counter=$metric/sorted($tags)
# 如果上报的数据带有tag，访问方式是这样的，比如：
curl http://127.0.0.1:6081/history/host01/qps/module=judge,project=falcon
```

