<!-- toc -->

# Task

Task is a necessary auxiliary module in monitor system. Timed task makes serevel features possible:

+ Index update, including total update of the index of diagram and the removal of junk index
+ Self data collect of the status of Falcon service modules, including the internal status data of Transfer, Gragh and Task module
+ Self checking task of Falcon


## Source Code Compilation

```bash
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/task
go get ./...
./control build
./control pack
```

The last step will generate an installation pack for service deployment.

## Service Deployment
Service deployment includes configuration changes, starting the service, testing the service, stopping the service etc. Before this, you need to unzip the the installation package to the deployment directory of the service.

```bash
# Change the configuration (the meaning of each setting is as follow)
mv cfg.example.json cfg.json
vim cfg.json

# Start the service
./control start

# Check the service (provided service open the 8002 http monitor port, the result shows the service started correctly)
curl -s "127.0.0.1:8002/health"

...
# Stop the service
./control stop

```

You can check the status of the service in the log after it is started. The log file address is "./var/app.log"。And you can also check the internal status data of the server by debugging the script  ```./test/debug```. For example, run ```bash ./test/debug``` and you can get the information of internal status of the server.


## Deployment Information
The configuration file is "./cfg.json" and there will be an example configuration file "cfg.example.json" in the installation pack. The meaning of each item in configuration is as follows.

```
debug: true/false, log will print debug information if it is true

http
    - enable: true/false, shows whether should open this http port; this port is a console port for sending control command, statistical command, debug command and so on
    - listen: port that http-server monitors

index
    - enable: true/false, shows whether index update is enabled
    - dsn: MySQL connection information of index service, default username is root and the password is null, host  127.0.0.1 and database is Graph (change if necessary)   
    - maxIdle: setting of MySQL connection pool, limiting the quantity of maximum idle connections, which better remains default
    - cluster: description of timed task of index update in back-end, in form of "Graph address: description of execution cycle"；different execution cycle settings for balancing overload in time dimension
        eg. two Graph instances are deployed in back-end and the configuration of cluster can be
        "cluster":{
            "test.hostname01:6071" : "0 0 0 ? * 0-5",   //week 0-5, daily index total update starts at 00:00:00;"0 0 0 ? * 0-5" is the expression of quartz
            "test.hostname02:6071" : "0 30 0 ? * 0-5",  //week 0-5, daily index total update starts at 00:00:00
        }
    - autoDelete: true|false, whether delete junk index automatically; default setting is false
    
collector
    - enable: true/false, whether self data collecting task of the status of Falcon is enabled
    - destUrl: push address of monitor data; default setting is 1988 port of this machine
    - srcUrlFmt: URL format of monitor data collecting; %s will be substituted with machine name or domain name
    - cluster: service list of Falcon in back-end in form of "module,hostname:port"; "module" can be Graph, Transfer, Task and so on

```

## Supplementary Notes

Please changes the configuration of collector after the Task module is deployed, so that Task can correctly collect the internal status of Transfer and Graph. Please the the configuration of monitor, so that Task module can self-check each module of Open-Falon (currently including  Transfer, Graph, Query, Judge etc.)

### About Self-Monitor Alarm
In need of multi-spot monitoring, we removed the self-monitor alarm feature from Task module since v0.0.10. For information of self-monitor of  Open-Falcon, please visit [here](http://book.open-falcon.org/zh/practice/monitor.html)。

### About Deleting Out-Dated Index
After monitor data is stopped from reporting, corresponding index will stop updating as well, becoming out-dated index that confuses users. So some users hope that we delete out-dated index.

Our original plan is: Index with data  reported is updated once a day through Task module and index that is not updated in the past 7 days will be deleted. However, many users cannot correctly configure the http port of Graph instance, then the index that normally reports monitor data cannot be updated. As a result, the legal index is mistakenly deleted by Task module. 

To solve the issue above, we removed the feature of automatically deleting the out-dated index in the default configuration of Task module (autoDelete=false)；you can turn it on if you are sure that your configuration of "index.cluster" is correct.

Of course, we came up with a safer method that deletes out-dated index manually, but users are required to trigger deletion of out-dated index that they desire. Here are the steps: 

1. Run a full update of index data, in other words, run  ```curl -s "127.0.0.1:6071/index/updateAll"``` in every Graph instance, then the full update of Graph instance is triggered asynchronously(provided the http monitor port of Graph is 6071). Proceed to step two when the full update of index in every Graph instance is finished. Normally, it costs no more than 30 minutes due to the quantity of counter and the performance of MySQL database.   

2. Start deleting out-dated index after the full update of index is finished  ``` curl -s "$Hostname.Of.Task:$Http.Port/index/delete" ```. Please make sure **the full update of index is finished** before index deletion.
