# Graph

graph是存储绘图数据的组件。graph组件 接收transfer组件推送上来的监控数据，同时处理query组件的查询请求、返回绘图数据。

## 源码编译

```bash
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/graph
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的安装包，拿着这个包去部署服务即可。

## 服务部署
服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```bash
# 修改配置, 配置项含义见下文
mv cfg.example.json cfg.json
vim cfg.json

# 启动服务
./control start

# 校验服务,这里假定服务开启了6071的http监听端口。检验结果为ok表明服务正常启动。
curl -s "127.0.0.1:6071/health"

...
# 停止服务
./control stop

```
启动服务后，可以通过日志查看服务的运行状态，日志文件地址为./var/app.log；如果需要详细的日志，可以将配置项debug设置为true。可以通过调试脚本```./test/debug```查看服务器的内部状态数据，如 运行 ```bash ./test/debug``` 可以得到服务器内部状态的统计信息。

## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```python
{
    "debug": false, //true or false, 是否开启debug日志
    "http": {
        "enabled": true, //true or false, 表示是否开启该http端口，该端口为控制端口，主要用来对graph发送控制命令、统计命令、debug命令
        "listen": "0.0.0.0:6071" //表示监听的http端口
    },
    "rpc": {
        "enabled": true, //true or false, 表示是否开启该rpc端口，该端口为数据接收端口
        "listen": "0.0.0.0:6070" //表示监听的rpc端口
    },
    "rrd": {
        "storage": "/home/work/data/6070" //绝对路径，历史数据的文件存储路径（如有必要，请修改为合适的路）
    },
    "db": {
        "dsn": "root:@tcp(127.0.0.1:3306)/graph?loc=Local&parseTime=true", //MySQL的连接信息，默认用户名是root，密码为空，host为127.0.0.1，database为graph（如有必要，请修改)
        "maxIdle": 4  //MySQL连接池配置，连接池允许的最大连接数，保持默认即可
    },
    "callTimeout": 5000,  //RPC调用超时时间，单位ms
    "migrate": {  //扩容graph时历史数据自动迁移
        "enabled": false,  //true or false, 表示graph是否处于数据迁移状态
        "concurrency": 2, //数据迁移时的并发连接数，建议保持默认
        "replicas": 500, //这是一致性hash算法需要的节点副本数量，建议不要变更，保持默认即可（必须和transfer的配置中保持一致）
        "cluster": { //未扩容前老的graph实例列表
            "graph-00" : "127.0.0.1:6070"
        }
    }
}

```

## 补充说明
部署完graph组件后，请修改transfer和query的配置，使这两个组件可以寻址到graph。
