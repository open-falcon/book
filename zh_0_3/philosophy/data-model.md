<!-- toc -->

# Data model

Open-Falcon，采用和OpenTSDB相似的数据格式：metric、endpoint加多组key value tags，举两个例子：

```bash
{
    metric: load.1min,
    endpoint: open-falcon-host,
    tags: srv=falcon,idc=aws-sgp,group=az1,
    value: 1.5,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
{
    metric: net.port.listen,
    endpoint: open-falcon-host,
    tags: port=3306,
    value: 1,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
```

其中，metric是监控指标名称，endpoint是监控实体，tags是监控数据的属性标签，counterType是Open-Falcon定义的数据类型(取值为GAUGE、COUNTER)，step为监控数据的上报周期，value和timestamp是有效的监控数据。
