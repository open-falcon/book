<!-- toc -->

# 数据收集相关问题
Open-Falcon数据收集，分为[绘图数据]收集和[报警数据]收集。下面介绍，如何验证两个链路的数据收集是否正常。


### 如何验证[绘图数据]收集是否正常
数据链路是：`agent->transfer->graph->query->dashboard`。graph有一个http接口可以验证`agent->transfer->graph`这条链路，比如graph的http端口是6071，可以这么访问验证：

```bash
# $endpoint和$counter是变量
curl http://127.0.0.1:6071/history/$endpoint/$counter

# 如果上报的数据不带tags，访问方式是这样的:
curl http://127.0.0.1:6071/history/host01/agent.alive

# 如果上报的数据带有tags，访问方式如下，其中tags为module=graph,project=falcon
curl http://127.0.0.1:6071/history/host01/qps/module=graph,project=falcon
```
如果调用上述接口返回空值，则说明agent没有上报数据、或者transfer服务异常。


### 如何验证[报警数据]收集是否正常

数据链路是：`agent->transfer->judge`，judge有一个http接口可以验证`agent->transfer->judge`这条链路，比如judge的http端口是6081，可以这么访问验证：

```bash
curl http://127.0.0.1:6081/history/$endpoint/$counter

# $endpoint和$counter是变量，举个例子：
curl http://127.0.0.1:6081/history/host01/cpu.idle

# counter=$metric/sorted($tags)
# 如果上报的数据带有tag，访问方式是这样的，比如：
curl http://127.0.0.1:6081/history/host01/qps/module=judge,project=falcon
```
如果调用上述接口返回空值，则说明agent没有上报数据、或者transfer服务异常。

