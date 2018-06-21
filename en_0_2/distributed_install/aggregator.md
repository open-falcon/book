<!-- toc -->

# Aggregator

Cluster aggregation module aggregates the value of one specified index of all machines in one cluster, providing a monitoring experience with cluster perspective. 


## Service Deployment
Service deployment includes configuration changes, starting the service, testing the service, stopping the service etc. Before this, you need to unzip installation package to  deployment directory of the service.

```
# Change the configuration (the meaning of each setting is as follow)
mv cfg.example.json cfg.json
vim cfg.json

# Start the service
./open-falcon start aggregator

# Check the log
./open-falcon monitor aggregator

# Stop the service
./open-falcon stop aggregator

```


## Configuraion Informaion
The configuration file is "./cfg.json" and there will be an example configuration file "cfg.example.json" in each installation package by default. The meaning of each setting is as follows

```
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6055"
    },
    "database": {
        "addr": "root:@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true",
        "idle": 10,
        "ids": [1, -1],
        "interval": 55
    },
    "api": {
        "connect_timeout": 500,
        "request_timeout": 2000,
        "plus_api": "http://127.0.0.1:8080",  #address where falcon-plus api module is running
        "plus_api_token": "default-token-used-in-server-side", #token used in mutual authentication with falcon-plus api module
        "push_api": "http://127.0.0.1:1988/v1/push"  #http port of push's data provided by Agent
    }
}

       
```
