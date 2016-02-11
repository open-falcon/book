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
query需要两个配置文件。一个是常规的配置文件./cfg.json，另一个是graph组件的实例配置(用于一致性哈希)。./cfg.json各配置项配的含义，如下

```
log_level: 可选 error/warn/info/debug/trace，默认为info

slow_log: 单位是毫秒，query的时候，较慢的请求，会被打印到日志中，默认是2000ms

debug: true/false, 如果为true，日志中会打印debug信息

http
    - enable: true/false, 表示是否开启该http端口，该端口为数据查询接口（即提供http api给用户查询数据）
    - listen: 表示监听的http端口

graph
    - backends: 后端graph列表文件，格式参考下面的介绍，该文件默认是./graph_backends.txt
    - reload_interval: 单位是秒，表示每隔多久自动reload一次backends列表文件中的内容
    - timeout: 单位是毫秒，表示和后端graph组件交互的超时时间，可以根据网络质量微调，建议保持默认
    - pingMethod: 后端提供的ping接口，用来探测连接是否可用，必须保持默认
    - max_conns: 连接池相关配置，最大连接数，建议保持默认
    - max_idle: 连接池相关配置，最大空闲连接数，建议保持默认
    - replicas: 这是一致性hash算法需要的节点副本数量，建议不要变更，保持默认即可

drrs #启用此功能前请确保DRRS已被正确安装配置（https://github.com/jdjr/drrs），不能与graph同时使用
	- enable: true/false, 表示是否开启向DRRS发送数据 #不能和graph的enable同时为true
	- useZk: 是否配置了zookeeper，若DRRS配置了多台master节点并且配置了zk，则配置为true
	- dest: DRRS中master节点的地址，若没有配置zk，则这里需要配置master节点的地址，格式为ip:port
	- replicas: 这是一致性hash算法需要的节点副本数量，建议不要变更，保持默认即可
	- maxIdle: 连接池相关配置，最大空闲连接数，建议保持默认
	- zk: zookeeper的相关配置信息，若useZk设置为true，则需要配置以下信息
		-ip: zk的ip地址，zk的端口需要保持默认的2181
		-addr: zk中DRRS配置信息的存放位置
		-timeout: zk的超时时间
  
```

graph组件实例配置文件，默认为./graph_backends.txt。其格式可以描述为

1.每行由空格分割的两列组成，第一列表示graph的名字，第二列表示具体的hostname:port
2.该文件需要和transfer配置文件中的cluster的配置项，保持一致

```
$ cat ./graph_backends.txt
graph-00 127.0.0.1:6070
graph-01 127.0.0.2:6070
```

## 补充说明
部署完成query组件后，请修改dashboard组件的配置、使其能够正确寻址到query组件。请确保query组件的graph列表(backends文件中的列表)配置是正确的。

DRRS为京东金融集团杭州研发团队的同事开发的一个轻量级的分布式环形数据服务组件，用于监控数据的持久化和绘图。该组件作用于graph组件类似，并且能够在保证绘图效率的前提下实现秒级扩容。DRRS组件与Graph组件无法同时使用。关于DRRS的内容请参考官方Github：[https://github.com/jdjr/drrs](https://github.com/jdjr/drrs)