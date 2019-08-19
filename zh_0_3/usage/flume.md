<!-- toc -->

# Flume监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Flume的数据采集可以通过脚本[flume-monitor](https://github.com/mdh67899/openfalcon-monitor-scripts/tree/master/flume)来做。

## 工作原理
```flume-monitor.py```是一个采集脚本，只需要放到falcon-agent的plugin目录，在portal中将对应的plugin绑定到主机组，falcon-agent会主动执行```flume-monitor.py```脚本，```flume-monitor.py```脚本执行结束后会输出json格式数据，由falcon-agent读取和解析数据

Flume运行时需要在配置文件中加入java环境变量，启动成功之后flume进程会监听一个端口，可以通过http请求的方式来抓取flume提供的metrics，```flume-monitor.py```脚本中配置了需要抓取的Flume组件metric，通过http的方式从flume端口中抓取需要的组件信息，输出json格式数据

比如我们在单台机器上部署了3个flume实例，可以在将脚本复制三份，改一下脚本中的```http url```地址，与flume监听的http端口一一对应，在portal中绑定好插件即可
