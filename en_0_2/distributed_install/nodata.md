<!-- toc -->

# Nodata

Nodata is used for monitoring errors in monitor data sending. nodata module and nad Judge module  work collaboratively. A piece of default mock data is generated when certain monitor item does not send data in time  with Nodata module. Users can configure alarm stratrgy in order that alarm goes off when mock data is received. Error detection in monitor data sending, served as a necessary complement of Judge module, provides real-time alarm with better reliablity and perfection.

## Service Deployment
Service deployment includes configuration changes, starting the service, testing the service, stopping the service etc. Before this, you need to unzip the the installation package to deployment directory of the service.

```
# Change the configuration (the meaning of each setting is as follow)
mv cfg.example.json cfg.json
vim cfg.json

# Start the service
./open-falcon start nodata

# Stop the service
./open-falcon stop nodata

# Check the log
./open-falcon monitor nodata

```

## Configuraion Informaion
The configuration file is "./cfg.json" and there will be an example configuration file "cfg.example.json" in each installation package by default. The meaning of each setting is as follows.

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
        "addr": "http://127.0.0.1:8080",  #address where Falcon-plus api module is running
        "token": "default-token-used-in-server-side"  #token used for communication authentication with Falcon-Plus API module
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
        "transferAddr": "127.0.0.1:6060",  #monitor address of Transfer, normally in form of "domain.transfer.service:6060"
        "batch": 500
    }
}
       
```
