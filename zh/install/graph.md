# Graph

graph是存储绘图数据的组件。graph组件 接收transfer组件推送上来的监控数据，同时处理query组件的查询请求、返回绘图数据。

## 源码编译

```bash
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
通过日志可以查看服务的运行状态，日志文件地址为./var/app.log。如果需要详细的日志，可以将配置项debug设置为true。

通过调试脚本```./test/debug```可以看服务器的内部状态数据，如 运行 ```bash ./test/debug``` 可以得到服务器内部状态的统计信息。

## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```
pid: 绝对路径，进程启动后的pid文件，这对于graph的平滑重启很重要（如有必要，请修改）

log: error/warn/info/debug/trace, 默认为info

debug: true/false, 如果为true，配合debugChecksum一起使用

debugChecksum: 如果debug为true，那么符合该checksum的counter，整个处理过程会在日志文件中详细打印，主要用于debug和排错

http
    - enable: true/false, 表示是否开启该http端口，该端口为控制端口，主要用来对graph发送控制命令、统计命令、debug命令等
    - listen: 表示监听的http端口

rpc
    - enable: true/false, 表示是否开启该rpc端口，该端口为数据接收端口
    - listen: 表示监听的http端口

rrd
    - storage: 绝对路径，历史数据的文件存储路径（如有必要，请修改为合适的路径）

db
    - dsn: MySQL的连接信息，默认用户名是root，密码为空，host为127.0.0.1，database为graph（如有必要，请修改）
    - maxIdle: MySQL连接池配置，连接池允许的最大连接数，保持默认即可

```

