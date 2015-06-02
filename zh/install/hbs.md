# HBS(Heartbeat Server)

心跳服务器，公司所有agent都会连到HBS，每分钟发一次心跳请求。

## 设计初衷

Portal的数据库中有一个host表，维护了公司所有机器的信息，比如hostname、ip等等。这个表中的数据通常是从公司CMDB中同步过来的。但是有些规模小一些的公司是没有CMDB的，那此时就需要手工往host表中录入数据，这很麻烦。于是我们赋予了HBS第一个功能：agent发送心跳信息给HBS的时候，会把hostname、ip、agent version、plugin version等信息告诉HBS，HBS负责更新host表。

falcon-agent有一个很大的特点，就是自发现，不用配置它应该采集什么数据，就自动去采集了。比如cpu、内存、磁盘、网卡流量等等都会自动采集。我们除了要采集这些基础信息之外，还需要做端口存活监控和进程数监控。那我们是否也要自动采集监听的端口和各个进程数目呢？我们没有这么做，因为这个数据量比较大，汇报上去之后用户大部分都是不关心的，太浪费。于是我们换了一个方式，只采集用户配置的。比如用户配置了对某个机器80端口的监控，我们才会去采集这个机器80端口的存活性。那agent如何知道自己应该采集哪些端口和进程呢？向HBS要，HBS去读取Portal的数据库，返回给agent。

之后我们会介绍一个用于判断报警的组件：Judge，Judge需要获取所有的报警策略，让Judge去读取Portal的DB么？不太好。因为Judge的实例数目比较多，如果公司有几十万机器，Judge实例数目可能会是几百个，几百个Judge实例去访问Portal数据库，也是一个比较大的压力。既然HBS无论如何都要访问Portal的数据库了，那就让HBS去获取所有的报警策略缓存在内存里，然后Judge去向HBS请求。这样一来，对Portal DB的压力就会大大减小。

## 源码安装

```bash
cd $GOPATH/src/github.com/open-falcon/hbs
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的包，拿着这个包去部署即可。

## 部署说明

hbs是可以水平扩展的，至少部署两个实例以保证可用性。一般一个实例可以搞定5000台机器，所以说，如果公司有10万台机器，可以部署20个hbs实例，前面架设lvs，agent中就配置上lvs vip即可。

## 配置说明

```
{
    "debug": true,
    "database": "root:password@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true", # Portal的数据库地址
    "hosts": "", # portal数据库中有个host表，如果表中数据是从其他系统同步过来的，此时配置为sync，否则就维持默认，留空即可
    "maxIdle": 100,
    "listen": ":6030", # hbs监听的rpc地址
    "trustable": [""],
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6031" # hbs监听的http地址
    }
}
```

## 验证

访问/health接口验证hbs是否工作正常。

```bash
curl 127.0.0.1:6031/health
```

另外就是查看hbs的log，log在var目录下
