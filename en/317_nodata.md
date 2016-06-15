##Nodata

"Nodata" is used to detect whether monitoring data is reported normally. "Nodata" works together with the real time alarming "judge" module. The procedure is as follows: if the collecting item with "nodata" configured does not report data as scheduled, "nodata" will generate a piece of default analog data; users configure a corresponding alarm strategy which will generate alarms when receiving mock data. As a necessary supplement to the "judge" module, detecting whether the collecting item is reporting normally can make the real-time alarm function of "judge" more reliable and complete.

##Preparatory work

This section is for old users of Open-Falcon, and new users may ignore this section and skip to the source code compiling part. If you have already used Open-Felcon for a while, as only a nodata service is added this time, you need to complete the following work in turn:

* Make sure the mysql data sheet "falcon_portal.mockcfg" is already set up. Among which, falcon_portal is the mysql database of the "portal" module, and mockcfg is the data sheet used to store "nodata" configuration. Table creating statements of "mockcfg" is as follows:
* Make sure the "portal" module has been already updated. The UI support to "nodata" configuration is newly added in the "portal" module.
* Install the back end services of "nodata", which is the following part of this article.

```
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
##Source code compiling
```
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile nodata
cd $GOPATH/src/github.com/open-falcon/nodata
go get ./...
./control build
./control pack
```

A "tar.gz" installation package will be packed at the last step. We can deploy services with this package.

##Service deployment

Service deployment includes configuration modification, starting services, verifying services, stopping services, etc. Before this, the installation package should be extracted into the deployment directory of the service.

```
#  Please refer to the following for configuration modification and configuration item meanings# Start services
mv cfg.example.json cfg.json
vim cfg.json

# 启动服务
./control start

# Verify services. It is assumed that the service enables the http monitoring port of 6090. If the verifying result is OK, the service starts normally. 
curl -s "127.0.0.1:6090/health"

...
# Stop services
./control stop
```
After the service starts, the running state of the service can be viewed through logs which are located in ./var/app.log. You can debug the script ./scripts/debug to view the internal state data of the server. For example, run bash ./scripts/debug to get the statistical information of the server's internal states.

##Configuration instruction

The default configuration file is ./cfg.json. By default, there is a configuration file example named cfg.example.json in the installation package. The meaning of each configuration item is as follows:

```
## Configuration
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6090" #the http service monitoring address of "nodata"
    },
    "query":{ #the configuration related to the "query" module
        "connectTimeout": 5000, the time-out period of the http connection when querying data, in ms
        "requestTimeout": 30000, #the time-out period of the http demand processing when querying data, in ms 
        "queryAddr": "127.0.0.1:9966" #the http monitoring address of the "query" module, whose general form is "domain.query.service:9966"
    },
    "config": { #configuration information
        "enabled": true,
        "dsn": "root:passwd@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true&wait_timeout=604800", #he database connection information of "portal", and the default database is falcon_portal
        "maxIdle": 4 #the number of idle connections in the mysql connection pool
    },
    "collector":{ #the configuration related to "nodata" data collection
        "enabled": true,
        "batch": 200, #data pieces per data collection, and it is recommended to use default values 
        "concurrent": 10 #collection concurrency, and it is recommended to use default values
    },
    "sender":{ #the configuration related to "mock" data sent by "nodata" "enabled": true,
        "enabled": true,
        "connectTimeout": 5000, #the time-out period of the http connection when sending data, in ms 
        "requestTimeout": 30000, #the time-out period of the http request when sending data, in ms
        "transferAddr": "127.0.0.1:6060", #the http monitoring address of "transfer", whose normal form is "domain.transfer.service:6060"
        "batch": 500, #the number of monitoring data pieces included in each package of data when sending data 
        "block": { #nodata blocking setting
            "enabled": false, #Whether to enable the blocking function. The function is disabled by default
            "threshold": 32 #The upper threshold to trigger "nodata" blocking action. For data items with "nodata" configured, when the interrupt percentage of data reporting exceeds the threshold, "nodata" will block the sending of "mock" data
        }
    }
}
```
The blocking function is used to avoid massive "nodata" false alarms, please refer to here for details.







