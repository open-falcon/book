# Agent

agent是open-falcon在服务器上的代理，每台机器上都需要部署agent。agent会自动采集预先定义的采集项，每隔60秒push到transfer。

## 源码编译

```bash
cd $GOPATH/src/github.com/open-falcon/agent
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

# 校验服务,这里假定agent开启了1988的http监听端口。检验结果为ok表明服务正常启动。
curl -s "127.0.0.1:1988/health"

...
# 停止服务
./control stop

```
通过日志可以查看服务的运行状态，日志文件地址为./var/app.log。如果需要详细的日志，可以将配置项debug设置为true。

## 配置说明
配置文件默认为./cfg.json。默认情况下，tar.gz会有一个cfg.example.json的配置文件示例。配置文件内容为，

```
# 请勿直接拷贝此文本,因为go不支持带注释的json
{
    "debug": true,
    "hostname": "",
    "ip": "",
    # 插件采集配置,默认不开启
    "plugin": {
        "enabled": false,
        "dir": "./plugin",
        "git": "https://coding.net/ulricqin/plugin.git",
        "logs": "./logs"
    },
    # 心跳服务配置,默认不开启
    "heartbeat": {
        "enabled": false,
        "addr": "",
        "interval": 60,
        "timeout": 1000
    },
    # 转发服务配置, 默认开启
    "transfer": {
        "enabled": true,
        "addr": "127.0.0.1:8433",
        "interval": 60,
        "timeout": 1000
    },
    "http": {
        "enabled": true,
        "listen": ":1988"
    },
    "collector": {
        "ifacePrefix": ["eth", "em"]
    },
    "ignore": {
        "cpu.busy": true,
        "mem.swapfree": true
    }
}

```

