# Nodata

nodata用于检测监控数据的上报异常。nodata和实时报警judge模块协同工作，过程为: 配置了nodata的采集项超时未上报数据，nodata生成一条默认的模拟数据；用户配置相应的报警策略，收到mock数据就产生报警。采集项上报异常检测，作为judge模块的一个必要补充，能够使judge的实时报警功能更加可靠、完善。

## 准备工作
这一节是写给Open-Falcon老用户的，新用户请忽略本小节、直接跳到[源码编译](#源码编译)部分即可。如果你已经使用Open-Falcon有一段时间，本次只是新增加一个nodata服务，那么你需要依次完成如下工作:

+ 确保已经建立mysql数据表falcon_portal.mockcfg。其中，[falcon_portal](https://github.com/open-falcon/scripts/blob/master/db_schema/portal-db-schema.sql)为portal组件的mysql数据库，mockcfg为存放nodata配置的数据表。mockcfg的建表语句，如下。
+ 确保已经更新了portal组件。portal组件中，新增了对nodata配置的UI支持。
+ 安装nodata后端服务。即本文的[后续部分](#源码编译)。

```sql
USE falcon_portal;
SET NAMES 'utf8';
 
/**
 * nodata mock config
 */
DROP TABLE IF EXISTS `mockcfg`;
CREATE TABLE `mockcfg` (
  `id`       BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
  `name`     VARCHAR(255) NOT NULL DEFAULT '' COMMENT 'name of mockcfg, used for uuid',
  `obj`      VARCHAR(10240) NOT NULL DEFAULT '' COMMENT 'desc of object',
  `obj_type` VARCHAR(255) NOT NULL DEFAULT '' COMMENT 'type of object, host or group or other',
  `metric`   VARCHAR(128) NOT NULL DEFAULT '',
  `tags`     VARCHAR(1024) NOT NULL DEFAULT '',
  `dstype`   VARCHAR(32)  NOT NULL DEFAULT 'GAUGE',
  `step`     INT(11) UNSIGNED  NOT NULL DEFAULT 60,
  `mock`     DOUBLE  NOT NULL DEFAULT 0  COMMENT 'mocked value when nodata occurs',
  `creator`  VARCHAR(64)  NOT NULL DEFAULT '',
  `t_create` DATETIME NOT NULL COMMENT 'create time',
  `t_modify` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'last modify time',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

```

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
    "config": { #配置信息
        "enabled": true,
        "dsn": "root:passwd@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true&wait_timeout=604800", #portal的数据库连接信息,默认数据库为falcon_portal
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
阻塞功能，是为了避免大规模的nodata误报警，详情请移步[这里](https://github.com/nieanan/nodata#阻塞设置)。
