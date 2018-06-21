<!-- toc -->

# Data model

The data model in Open-Falcon is similar to the one in OpenTSDB：metric and endpoint with a couple of key value tags. Here are two examples:

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

In those two examples, metric is the name of monitor index, endpoint is the object that is being monitoed, tags are about the attributes the monitor data, counterType is the data type defined by Open-Falcon (which can be GAUGE or COUNTER), step is the reporting cycle of monitor data, and value and timestamp arw valid monitor data.