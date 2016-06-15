##Aggregator

The cluster aggregation module is used to aggregate the values of an indicator of all machines under a cluster to provide a monitoring experience from the cluster view.

##Preparatory work

If you have already installed "open-falcon", please check:
whether the following code is included in your portal: https://github.com/open-falcon/portal/blob/master/web/model/cluster.py your "portal". If the answer is yes, the version is OK; Otherwise, the original "portal" needs to be upgraded to the latest version.

A new sheet is added into the falcon_portal database.

```
USE falcon_portal;
SET NAMES 'utf8';

DROP TABLE IF EXISTS cluster;
CREATE TABLE cluster
(
  id          INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  grp_id      INT            NOT NULL,
  numerator   VARCHAR(10240) NOT NULL,
  denominator VARCHAR(10240) NOT NULL,
  endpoint    VARCHAR(255)   NOT NULL,
  metric      VARCHAR(255)   NOT NULL,
  tags        VARCHAR(255)   NOT NULL,
  ds_type     VARCHAR(255)   NOT NULL,
  step        INT            NOT NULL,
  last_update TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  creator     VARCHAR(255)   NOT NULL,
  PRIMARY KEY (id)
)
  ENGINE =InnoDB
  DEFAULT CHARSET =latin1;
  ```
  ##Source code compiling
  
  ```
cd $GOPATH/src/github.com/open-falcon
git clone https://github.com/open-falcon/sdk.git
git clone https://github.com/open-falcon/aggregator.git
cd aggregator
go get ./...
./control build
./control pack
```
  
A "tar.gz" installation package will be packed at the last step. We can deploy services with this package.

##Service deployment

Service deployment includes configuration modification, starting services, verifying services, stopping services, etc. Before this, the installation package should be extracted into the deployment directory of the service.

```
# Please refer to the following for configuration modification and configuration item meanings
mv cfg.example.json cfg.json
vim cfg.json

# Start services
./control start

# Verify services to ensure whether the port is being monitored 
ss -tln

# Examine logs
./control tail

...
# Stop services
./control stop
```
##Configuration instruction

The default configuration file is ./cfg.json. By default, there is a configuration file example named cfg.example.json in the installation package. The meaning of each configuration item is as follows:

```
## Configuration
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6055"
    },
    "database": {
        "addr": "root:@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true",
        "idle": 10,
        "ids": [1,-1], # More than one instance can be deployed in the "aggregator" module. This configuration means the ID scope of the cluster in the database which the current instance is dealing with.
        "interval": 55
    },
    "api": {
        "hostnames": "http://127.0.0.1:5050/api/group/%s/hosts.json", # It is required to modify into the ip:port of your portal
        "push": "http://127.0.0.1:6060/api/push", # It is required to modify into the ip:port of your "transfer"
        "graphLast": "http://127.0.0.1:9966/graph/last" # It is required to modify into the ip:port of your "query"
    }
}
```


  