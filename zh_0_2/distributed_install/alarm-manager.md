<!-- toc -->

# Alarm-Manager
告警管理模块，该模块主要有两大功能，一个用于注册告警数据接收服务、提供告警事件多条件查询功能；另外一个是针对告警事件升级为故障，定制化处理功能。第一版仅提供api，详细参考api接口：[api v0.2](http://api.open-falcon.com/)

## 设计初衷
面对企业级的监控体系，每天避免不了大量的告警，当前缺少对告警的管理机制，而且大量告警产生时伴随告警风暴，往往造成告警处理记录缺失、以及需要进行故障总结等问题。告警管理，通过**指定接收者和其他条件**可以快速过滤你关注的告警、告警数量、已添加至故障的告警信息等，同时针对告警信息可以生成一个跟进事件(事件定义为故障)，将事件的处理过程通过时间轴管理起来，目前实现了对故障的新增、更改、增加告警事件、关注、状态变更(关闭、废弃、重新打开)、输出故障时间轴等功能。后续的版本，对于故障的时间轴统计故障的MTTA/MTTR(即平均响应时间和平均恢复时间)，自动生成故障总结、实现告警信息的聚合、告警关联，api接口接入前端这些功能都可以有。

## 部署说明
服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```
# 修改配置, 配置项含义见下文, 注意graph集群的配置
mv cfg.example.json cfg.json
vim cfg.json

# 启动
./open-falcon start alarm-manager

# 停止
./open-falcon stop alarm-manager

# 查看日志
./open-falcon monitor alarm-manager
```

## 配置说明
配置文件必须叫cfg.json，可以基于cfg.example.json修改
```
{
    "log_level":"debug", 
    "db":{
        "am":"root:Mm123456mM@tcp(10.38.163.95:13308)/am?charset=utf8&parseTime=True&loc=Local",
        "db_bug": true
    },
    "api": {
        "plus_api":"http://127.0.0.1:8080",   //falcon-plus api模块的运行地址
        "plus_api_token": "default-token-used-in-server-side" //用于和falcon-plus api模块服务端之间的通信认证token              
    }, 
    "listen": ":9922"
}
```

## 数据来源
目前告警管理数据来源于alarm组件推的方式。alarm-manager组件实现告警数据的自管理，组件提供专门的接收数据接口，其它告警数据通道可按照一定数据格式进行推动，用于其它监控平台的数据流入、告警管理。