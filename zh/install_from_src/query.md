# Query
query组件，提供统一的绘图数据查询入口。query组件接收查询请求，根据一致性哈希算法去相应的graph实例查询不同metric的数据，然后汇总拿到的数据，最后统一返回给用户。

## 源码编译

```bash
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/query
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的安装包，拿着这个包去部署服务即可。

## 服务部署
服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```bash
# 修改配置, 配置项含义见下文, 注意graph集群的配置
mv cfg.example.json cfg.json
vim cfg.json

## 修改graph集群配置, 默认在./graph_backends.txt中定义
vim graph_backends.txt


# 启动服务
./control start

# 校验服务,这里假定服务开启了9966的http监听端口。检验结果为ok表明服务正常启动。
curl -s "127.0.0.1:9966/health"

...
# 停止服务
./control stop

```
服务启动后，可以通过日志查看服务的运行状态，日志文件地址为./var/app.log。可以通过查询脚本```./scripts/query```读取绘图数据，如 运行 ```bash ./scripts/query "ur.endpoint" "ur.counter"```可以查询Endpoint="ur.endpoint" & Counter="ur.counter"对应的绘图数据。

## 配置说明

注意: 请确保 `graph.replicas`和`graph.cluster` 的内容与transfer的配置**完全一致**

```bash
{
    "debug": "false",   // 是否开启debug日志
    "http": {
        "enabled":  true,          // 是否开启http.server
        "listen":   "0.0.0.0:9966" // http.server监听地址&端口
    },
    "graph": {
        "connTimeout": 1000, // 单位是毫秒，与后端graph建立连接的超时时间，可以根据网络质量微调，建议保持默认
        "callTimeout": 5000, // 单位是毫秒，从后端graph读取数据的超时时间，可以根据网络质量微调，建议保持默认
        "maxConns": 32,      // 连接池相关配置，最大连接数，建议保持默认
        "maxIdle": 32,       // 连接池相关配置，最大空闲连接数，建议保持默认
        "replicas": 500,     // 这是一致性hash算法需要的节点副本数量，应该与transfer配置保持一致
        "cluster": {         // 后端的graph列表，应该与transfer配置保持一致；不支持一条记录中配置两个地址
            "graph-00": "test.hostname01:6070",
            "graph-01": "test.hostname02:6070"
        },
        "api": {  // 适配grafana需要的API配置
            "query": "http://127.0.0.1:9966",     // query的http地址
            "dashboard": "http://127.0.0.1:8081", // dashboard的http地址
            "max": 500                            //API返回结果的最大数量
        }
    }
}
```

## 补充说明
部署完成query组件后，请修改dashboard组件的配置、使其能够正确寻址到query组件。请确保query组件的graph列表 与 transfer的配置 一致。
