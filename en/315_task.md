##Task

"Task" is a necessary auxiliary module in a monitoring system. The following functions can be implemented through the timed task function:

* Index updates, including the full quantity updates of chart indexes and spamdexing cleaning.
* Data collection of "falcon" service components' own status. The timed task function can collect the internal status data of the three services of "transfer", "graph" and "task".
* Self-examining and control tasks of "falcon".

##Source code compiling

We provide the release package of the module, which can be downloaded and deployed directly. Or, you can also implement source code compiling as the following method:

```
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/task
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

# Verify services. It is assumed that the service enables the http monitoring port of 8002. If the verifying result is OK, the service starts normally.
curl -s "127.0.0.1:8002/health"

...
# Stop services
./control stop
```
After the service starts, the running status of the service can be viewed through logs which are located in ./var/app.log. You can debug the script ./test/debug to view the internal state data of the server. For example, run bash ./test/debug to get the statistical information of the server's internal states.

##Configuration instruction

The default configuration file is ./cfg.json. By default, there is a configuration file example named cfg.example.json in the installation package. The meaning of each configuration item is as follows:
```
debug: true/false, 如果为true，If it is true, debug information will be printed in the log

http
    - enable: true/false, ndicates whether to enable the http port. The http port is a control port, and mainly used to send control commands, statistics commands, debug commands and other commands for the "task" component.
    - listen: indicates the port monitored by the http-server.

index
    - enable: true/false,indicates whether to enable the index update task
    - dsn: he connection information of MySQL of index services. The default user name is root, the default password is empty, the default host is 127.0.0.1, and the default database is graph (modify if necessary)
    - maxIdle: the connection pool configuration of MySQL, the allowed max number of free connections of the connection pool, which can be maintained as default
    - cluster: the timed task description of a back end graph index. A record can be described as follows: "graph address: the description of the execution cycle". The load balance in time can be realized by setting different execution cycles.
        eg. two graph instances are deployed at the back end, and the cluster can be configured as:
        "cluster":{
            "test.hostname01:6071" : "0 0 0 ? * 0-5",   //Start executing index full quantity updates at 00:00:00 everyday from Sunday to Friday;"0 0 0 ? * 0-5" is the expression of quartz
            "test.hostname02:6071" : "0 30 0 ? * 0-5",  //Start executing index full quantity updates at 12:30:00 AM everyday from Sunday to Friday
        }
    - autoDelete: true|false, whether to delete spamdexing automatically. The default setting is "false"

collector
    - enable: true/false,  indicates whether to enable its own status collecting task of "falcon".
    - destUrl: the "push" address of monitoring data, and the default setting is the native 1988 interface
    - srcUrlFmt: the url format of monitoring data collection, and %s will be replaced by the machine name or the field name.
    - cluster: the back end service list of "falcon", which can be expressed by specific "module,hostname:port", and the value of "module" can be "graph", "transfer", "task", etc.
    ```
    
##Supplementary note
After deploying the "task" component, please modify the configuration of the "collector" component to make "task" be able to collect the internal status of "transfer & graph" correctly, and modify the configuration of the "monitor" component to make the "task" component be able to examine and control each component of Open-Falon (components, such as "transfer", "graph", "query" and "judge", are supported currently)

##About self-monitoring alarm

Because of the needs of multi-point monitoring, since the version of v0.0.10, self-monitoring alarm function has been removed from the "Task" module. Please refer to here for the details about Open-Falcon self-monitoring.

##About the clearing of expired indexes

After monitoring data reporting stops, the update of the indexes corresponding to the data will stop too, and the indexes will become expired. Some users wish to delete the expired indexes which can result in bad influences.

Our original scheme is: By the "task" module, indexes with data reporting should be updated once a day, while indexes that have not been updated for 7 days should be cleaned. However, many users can't configure the http interface of "graph" instances correctly, which causes that the indexes who report monitoring data normally can't be updated; After 7 days, the legal indexes will be deleted by the "task" module by mistake.

To resolve the above problem, we disable the function that the "task" module deletes expired indexes automatically (autoDelete=false) in the default configuration; If you are sure about the configured index.cluster, you can enable the function by yourself.

However, we provide a method that deletes expired indexes manually, which is safer. Users can trigger index deleting action as necessary, and specific procedures are as follows:

1. Implement a full quantity update for index data. The method is: for each graph instance, run curl -s "127.0.0.1:6071/index/updateAll" to trigger the index full quantity update of the graph instance (it is assumed that the http monitoring port of "graph" is 6071). After the index full quantity update of all graph instances is finished, the next step should be conducted. The time that the index full quantity update of a graph instance needs depends on the number of counters and the performance of mySQL database. Usually, the time does not exceed 30 min.
2. After the full quantity update of indexes finishes, delete expired indexes curl -s "$Hostname.Of.Task:$Http.Port/index/delete". Make sure the full quantity update of indexes has finished before deleting indexes.

