<!-- toc -->

prometheus作为优秀的开源监控产品，本身不仅完整的指标体系，还拥有丰富的指标采集解决方案。通过各种exporter可以覆盖中间件，操作系统，开发语言等等方面的监控指标采集

**对于在使用 [open-falcon](https://github.com/open-falcon/falcon-plus) 的用户，你也可以通过 [prometheus-exporter-collector](https://github.com/n9e/prometheus-exporter-collector) 将收集到的数据发送给 open-falcon。**

```
./prometheus-exporter-collector -h
Usage: ./prometheus-exporter-collector [-h] [-b backend] [-s step]

Options: 
  -b string
        send metrics to backend: n9e, falcon (default "n9e")
  -h    help
  -s int
        set default step of falcon metrics (default 60)
```
- `-b falcon`： 以 open-falcon 作为数据接收方
- `-s 60`: metric 的 step 设置为60s

**下面是一个具体的例子**：通过 prometheus-exporter-collector， 获取 redis-exporter 的metrics，并发送给 open-falcon。

### 1. 下载和编译 redis_exporter

```
git clone https://github.com/oliver006/redis_exporter.git
cd redis_exporter
go build .
./redis_exporter --version
./redis_exporter -redis.addr redis://127.0.0.1:6379

//注意，请先确保 redis 已成功运行在127.0.0.1:6379 上。
```

这样，就可以看到 redis_exporter 已经成功运行，并监听在 `:9121/metrics` 。
 
### 2. 运行 prometheus-exporter-collector 并发送数据给 open-falcon
- 检查prometheus-exporter-collector的配置文件，确保 `exporter_urls` 设置为 `http://127.0.0.1:9121/metrics`

```
$ cat plugin.test.json

{
  "exporter_urls": [
    "http://127.0.0.1:9121/metrics"
  ],
  "append_tags": ["region=bj", "dept=cloud"],
  "endpoint": "127.0.0.100",
  "ignore_metrics_prefix": ["go_"],
  "metric_prefix": "",
  "metric_type": {},
  "default_mapping_metric_type": "COUNTER",
  "timeout": 500
}
```

- 运行prometheus-exporter-collector，将输出发送给本机的 falcon-agent

```
cat plugin.test.json | ./prometheus-exporter-collector -b falcon -s 60 | curl -X POST -d @- http://127.0.0.1:1988/v1/push
```
