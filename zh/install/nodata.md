# Nodata

nodata用于检测监控数据的上报异常。nodata和实时报警judge模块协同工作，过程为: 配置了nodata的采集项超时未上报数据，nodata生成一条默认的模拟数据；用户配置相应的报警策略，收到mock数据就产生报警。采集项上报异常检测，作为judge模块的一个必要补充，能够使judge的实时报警功能更加可靠、完善。

## 源码编译

```bash
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile nodata
cd $GOPATH/src/github.com/open-falcon/nodata
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

# 校验服务,这里假定服务开启了6090的http监听端口。检验结果为ok表明服务正常启动。
curl -s "127.0.0.1:6090/health"

...
# 停止服务
./control stop

```
服务启动后，可以通过日志查看服务的运行状态，日志文件地址为./var/app.log。可以通过调试脚本```./scripts/debug```查看服务器的内部状态数据，如 运行 ```bash ./scripts/debug``` 可以得到服务器内部状态的统计信息。


## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```bash
## Configuration
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6090" #nodata的http服务监听地址
    },
    "query":{ #query组件相关的配置
        "connectTimeout": 5000, #查询数据时http连接超时时间,单位ms
        "requestTimeout": 30000, #查询数据时http请求处理超时时间,单位ms
        "queryAddr": "127.0.0.1:9966" #query组件的http监听地址,一般形如"domain.query.service:9966"
    },
    "config": { #nodata配置相关的信息
        "enabled": true,
        "dsn": "root:passwd@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true&wait_timeout=604800", #mysql数据库配置
        "maxIdle": 4 #mysql连接池空闲连接数
    },
    "collector":{ #nodata数据采集相关的配置
        "enabled": true,
        "batch": 200, #一次数据采集的条数,建议使用默认值
        "concurrent": 10 #采集并发度,建议使用默认值
    },
    "sender":{ #nodata发送mock数据相关的配置
        "enabled": true,
        "connectTimeout": 5000, #发送数据时http连接超时时间,单位ms
        "requestTimeout": 30000, #发送数据时http请求超时时间,单位ms
        "transferAddr": "127.0.0.1:6060", #transfer的http监听地址,一般形如"domain.transfer.service:6060"
        "batch": 500, #发送数据时,每包数据包含的监控数据条数
        "block": { #nodata阻塞设置
            "enabled": false, #是否开启阻塞功能.默认不开启此功能
            "threshold": 32 #触发nodata阻塞操作的阈值上限.当配置了nodata的数据项,数据上报中断的百分比,大于此阈值上限时,nodata阻塞mock数据的发送
        }
    }
}
       
```

