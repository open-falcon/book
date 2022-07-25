<!-- toc -->

# Aggregator

集群聚合模块。聚合某集群下的所有机器的某个指标的值，提供一种集群视角的监控体验。


## 服务部署
服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```
# 修改配置, 配置项含义见下文
mv cfg.example.json cfg.json
vim cfg.json

# 启动服务
./open-falcon start aggregator

# 检查log
./open-falcon monitor aggregator

# 停止服务
./open-falcon stop aggregator

```


## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6055"
    },
    "database": {
        "addr": "root:@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true", // 如数据库使用密码, 配置稍有不同: "addr": "root:password@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true&allowNativePasswords=true"
        "idle": 10,
        "ids": [1, -1],
        "interval": 55
    },
    "api": {
        "connect_timeout": 500,
        "request_timeout": 2000,
        "plus_api": "http://127.0.0.1:8080",  #falcon-plus api模块的运行地址
        "plus_api_token": "default-token-used-in-server-side", #和falcon-plus api 模块交互的认证token
        "push_api": "http://127.0.0.1:1988/v1/push"  #push数据的http接口，这是agent提供的接口
    }
}

       
```
