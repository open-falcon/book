# Transfer

transfer是数据转发服务。它接收agent上报的数据，然后按照哈希规则进行数据分片、并将分片后的数据分别push给graph&judge等组件。

## 源码编译

```bash
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/transfer
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

# 校验服务,这里假定服务开启了6060的http监听端口。检验结果为ok表明服务正常启动。
curl -s "127.0.0.1:6060/health"

...
# 停止服务
./control stop

```
服务启动后，可以通过日志查看服务的运行状态，日志文件地址为./var/app.log。可以通过调试脚本```./test/debug```查看服务器的内部状态数据，如 运行 ```bash ./test/debug``` 可以得到服务器内部状态的统计信息。


## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

## Configuration
```
    debug: true/false, 如果为true，日志中会打印debug信息

    http
        - enable: true/false, 表示是否开启该http端口，该端口为控制端口，主要用来对transfer发送控制命令、统计命令、debug命令等
        - listen: 表示监听的http端口

    rpc
        - enable: true/false, 表示是否开启该jsonrpc数据接收端口, Agent发送数据使用的就是该端口
        - listen: 表示监听的http端口

    socket #即将被废弃,请避免使用
        - enable: true/false, 表示是否开启该telnet方式的数据接收端口，这是为了方便用户一行行的发送数据给transfer
        - listen: 表示监听的http端口

    judge
        - enable: true/false, 表示是否开启向judge发送数据
        - batch: 数据转发的批量大小，可以加快发送速度，建议保持默认值
        - connTimeout: 单位是毫秒，与后端建立连接的超时时间，可以根据网络质量微调，建议保持默认
        - callTimeout: 单位是毫秒，发送数据给后端的超时时间，可以根据网络质量微调，建议保持默认
        - pingMethod: 后端提供的ping接口，用来探测连接是否可用，必须保持默认
        - maxConns: 连接池相关配置，最大连接数，建议保持默认
        - maxIdle: 连接池相关配置，最大空闲连接数，建议保持默认
        - replicas: 这是一致性hash算法需要的节点副本数量，建议不要变更，保持默认即可
        - cluster: key-value形式的字典，表示后端的judge列表，其中key代表后端judge名字，value代表的是具体的ip:port

    graph
        - enable: true/false, 表示是否开启向graph发送数据
        - batch: 数据转发的批量大小，可以加快发送速度，建议保持默认值
        - connTimeout: 单位是毫秒，与后端建立连接的超时时间，可以根据网络质量微调，建议保持默认
        - callTimeout: 单位是毫秒，发送数据给后端的超时时间，可以根据网络质量微调，建议保持默认
        - pingMethod: 后端提供的ping接口，用来探测连接是否可用，必须保持默认
        - maxConns: 连接池相关配置，最大连接数，建议保持默认
        - maxIdle: 连接池相关配置，最大空闲连接数，建议保持默认
        - replicas: 这是一致性hash算法需要的节点副本数量，建议不要变更，保持默认即可
        - cluster: key-value形式的字典，表示后端的graph列表，其中key代表后端graph名字，value代表的是具体的ip:port(多个地址用逗号隔开, transfer会将同一份数据发送至各个地址，利用这个特性可以实现数据的多重备份)

    tsdb
        - enabled: true/false, 表示是否开启向open tsdb发送数据
        - batch: 数据转发的批量大小，可以加快发送速度
        - connTimeout: 单位是毫秒，与后端建立连接的超时时间，可以根据网络质量微调，建议保持默认
        - callTimeout: 单位是毫秒，发送数据给后端的超时时间，可以根据网络质量微调，建议保持默认
        - maxConns: 连接池相关配置，最大连接数，建议保持默认
        - maxIdle: 连接池相关配置，最大空闲连接数，建议保持默认
        - retry: 连接后端的重试次数和发送数据的重试次数
        - address: tsdb地址或者tsdb集群vip地址, 通过tcp连接tsdb. 
       
```

## 补充说明
部署完成transfer组件后，请修改agent的配置，使其指向正确的transfer地址。在安装完graph和judge后，请修改transfer的相应配置、使其能够正确寻址到这两个组件。


## 视频教程

为transfer模块录制了一个视频，做了源码级解读：http://www.jikexueyuan.com/course/2061.html
