# Alarm

alarm模块是处理报警event的，judge产生的报警event写入redis，alarm从redis读取处理

## 设计初衷

报警event的处理逻辑并非仅仅是发邮件、发短信这么简单。为了能够自动化对event做处理，alarm需要支持在产生event的时候回调用户提供的接口；有的时候报警短信、邮件太多，对于优先级比较低的报警，希望做报警合并，这些逻辑都是在alarm中做的。

## 源码安装

```bash
cd $GOPATH/src/github.com/open-falcon/alarm
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的包，拿着这个包去部署即可。

## 部署说明

alarm是个单点。对于未恢复的告警是放到alarm的内存中的，alarm还需要做报警合并，故而alarm只能部署一个实例。后期需要想办法改进。

## 配置说明

```
{
    "debug": true,
    "uicToken": "",
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6060" # 未恢复的告警就是通过alarm的http页面来看的
    },
    "queue": {
        "sms": "/sms", # 需要与sender配置成相同的，维持默认即可
        "mail": "/mail"
    },
    "redis": {
        "addr": "127.0.0.1:6379", # 与judge、sender相同的redis地址
        "maxIdle": 5,
        "highQueues": [
            "event:p0",
            "event:p1"
        ],
        "lowQueues": [
            "event:p2",
            "event:p3",
            "event:p4",
            "event:p5",
            "event:p6"
        ],
        "userSmsQueue": "/queue/user/sms", # 这两个queue维持默认即可
        "userMailQueue": "/queue/user/mail"
    },
    "api": {
        "portal": "http://falcon.example.com", # 内网可访问的portal的地址
        "uic": "http://uic.example.com", # 内网可访问的uic(或fe)的地址
        "links": "http://link.example.com" # 外网可访问的links的地址
    }
}
```

api部分portal和uic可以配置成内网可访问的地址，速度比较快，但是links要配置成外网可访问的地址，注意喽

## 报警合并

如果某个核心服务挂了，可能会造成大面积报警，为了减少报警短信数量，我们做了报警合并功能。把报警信息写入links模块，然后links返回一个url地址给alarm，alarm将这个url链接发给用户，这样用户只要收到一条短信（里边是个url地址），点击url进去就是多条报警内容。

highQueues中配置的几个event队列中的事件是不会做报警合并的，因为那些是高优先级的报警，报警合并只是针对lowQueues中的事件。如果所有的事件都不想做报警合并，就把所有的event队列都配置到highQueues中即可

## 验证

log没问题就是没问题了，log在var目录

可以访问alarm的http地址，会展示一个web页面“未恢复的告警列表”

## 补充

alarm搭建完成了，我们可以回去修改fe的配置：falconAlarm
