## 环境准备

请参考[环境准备](./prepare.md)

同时，请再次检查当前的工作目录设置：

	export HOME=/home/work
	export WORKSPACE=$HOME/open-falcon
	mkdir -p $WORKSPACE

## 安装Transfer

transfer默认监听在:8433端口上，agent会通过jsonrpc的方式来push数据上来。

```bash
cd $WORKSPACE/transfer/
mv cfg.example.json cfg.json

# 默认情况下（所有组件都在同一台服务器上），保持cfg.json不变即可
# cfg.json中的各配置项，可以参考 https://github.com/open-falcon/transfer/blob/master/README.md
# 如有必要，请酌情修改cfg.json

# 启动transfer
./control start

# 校验服务,这里假定服务开启了6060的http监听端口。检验结果为ok表明服务正常启动。
curl -s "http://127.0.0.1:6060/health"

#查看日志
./control tail

# 停止transfer
./control stop
```

## 安装Agent

每台机器上，都需要部署agent，agent会自动采集预先定义的各种采集项，每隔60秒，push到transfer。

```bash
cd $WORKSPACE/agent/
mv cfg.example.json cfg.json

vim cfg.json
- 修改 transfer这个配置项的enabled为 true，表示开启向transfer发送数据的功能
- 修改 transfer这个配置项的addr为：["127.0.0.1:8433"] (改地址为transfer组件的监听地址, 为列表形式，可配置多个transfer实例的地址，用逗号分隔)

# 默认情况下（所有组件都在同一台服务器上），保持cfg.json不变即可
# cfg.json中的各配置项，可以参考 https://github.com/open-falcon/agent/blob/master/README.md

# 启动
./control start

# 查看日志
./control tail
```

## 安装Graph
graph组件是存储绘图数据、历史数据的组件。transfer会把接收到的数据，转发给graph。

```bash
cd $WORKSPACE/graph/
mv cfg.example.json cfg.json

# 默认情况下（所有组件都在同一台服务器上），保持cfg.json不变即可
# cfg.json中的各配置项，可以参考 https://github.com/open-falcon/graph/blob/master/README.md

# 启动
./control start

# 查看日志
./control tail

# 校验服务,这里假定服务开启了6071的http监听端口。检验结果为ok表明服务正常启动。
curl -s "http://127.0.0.1:6071/health"
```

## 安装Query
query组件，绘图数据的查询接口，query组件收到用户的查询请求后，会从后端的多个graph，查询相应的数据，聚合后，再返回给用户。

```bash
cd $WORKSPACE/query/
mv cfg.example.json cfg.json

# 默认情况下（所有组件都在同一台服务器上），保持cfg.json不变即可
# cfg.json中的各配置项，可以参考 https://github.com/open-falcon/query/blob/master/README.md

# 启动
./control start

# 查看日志
./control tail
```

## 安装Dashboard
dashboard是面向用户的查询界面，在这里，用户可以看到push到graph中的所有数据，并查看其趋势图。

**Install dependency**
```bash
yum install -y python-virtualenv mysql-devel  # run as root

cd $WORKSPACE/dashboard/
virtualenv ./env

./env/bin/pip install -r pip_requirements.txt
```

**Configuration**
```bash
# config的路径为 $WORKSPACE/dashboard/rrd/config.py，里面有数据库相关的配置信息，如有必要，请修改。默认情况下(所有组件都在同一台服务器上)，保持默认配置即可
# 数据库表结构初始化，请参考前面的 环境准备 阶段
```

**启动**

	./control start
	--> goto http://127.0.0.1:8081


**查看日志**

	./control tail

**停止**

	./control stop


**screenshots**

**首页**

![Homepage](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/falcon-homepage.png)

*在dashboard首页的endpoint字段中，搜索你的机器名，不出意外就可以看到上报的数据了*

**Screen**

![Screen](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/falcon-screen.png)

**大图**

![Big chart](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/falcon-big-chart.png)
