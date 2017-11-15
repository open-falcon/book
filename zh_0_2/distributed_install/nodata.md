<!-- toc -->

# Nodata

nodata用于检测监控数据的上报异常。nodata和实时报警judge模块协同工作，过程为: 配置了nodata的采集项超时未上报数据，nodata生成一条默认的模拟数据；用户配置相应的报警策略，收到mock数据就产生报警。采集项上报异常检测，作为judge模块的一个必要补充，能够使judge的实时报警功能更加可靠、完善。

## 服务部署
服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```
# 修改配置, 配置项含义见下文
mv cfg.example.json cfg.json
vim cfg.json

# 启动服务
./open-falcon start nodata

# 停止服务
./open-falcon stop nodata

# 检查日志
./open-falcon monitor nodata

```

## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6090"
    },
    "plus_api":{
        "connectTimeout": 500,
        "requestTimeout": 2000,
        "addr": "http://127.0.0.1:8080",  #falcon-plus api模块的运行地址
        "token": "default-token-used-in-server-side"  #用于和falcon-plus api模块的交互认证token
    },
    "config": {
        "enabled": true,
        "dsn": "root:@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true&wait_timeout=604800",
        "maxIdle": 4
    },
    "collector":{
        "enabled": true,
        "batch": 200,
        "concurrent": 10
    },
    "sender":{
        "enabled": true,
        "connectTimeout": 500,
        "requestTimeout": 2000,
        "transferAddr": "127.0.0.1:6060",  #transfer的http监听地址,一般形如"domain.transfer.service:6060"
        "batch": 500
    }
}
       
```
