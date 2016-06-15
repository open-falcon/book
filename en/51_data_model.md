# 5.1 Data Model

Data model
Open-Falcon uses the data format similar to OpenTSDB: metric, endpoint, plus multi-group key value tags. There are two examples:
```
{
    metric: load.1min,
    endpoint: open-Falcon-host,
    tags: srv=Falcon,idc=aws-sgp,group=az1,
    value: 1.5,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
{
    metric: net.port.listen,
    endpoint: open-Falcon-host,
    tags: port=3306,
    value: 1,
    timestamp: `date +%s`,
    counterType: GAUGE,
    step: 60
}
```
Among them, metric is the name of monitoring metrics, endpoint is the monitoring entity, tags is the attribute tag of monitoring data, counterType is the data type defined by Open-Falcon (the values are GAUGE, COUNTER), step is the reported period of monitoring data, value and timestamp is the valid monitoring data.