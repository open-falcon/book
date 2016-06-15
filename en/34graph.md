##Graph

Graph is the component that stores graphic data. The Graph component receives monitored data pushed by the Transfer component, and processes query requests of the Query component and returns graphic data.

##Source Code Compilation

```
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/graph
go get ./...
./control build
./control pack
```
A tar.gz package will be packed at the last step. We can deploy with this package.

##Service Deployment

Service deployment includes configuration modification, starting service, verifying service, stopping service, etc. Before service deployment, you need to unzip the installation package to the deployment directory of the service.

```
# Modify configuration. Please refer to the following for meanings of configuration items
mv cfg.example.json cfg.json
vim cfg.json

# tart service.
./control start

# Verify service. Here it is assumed that the service enables the 6071 http listening port. The verification result is ok, indicating that the service has started properly.。
curl -s "127.0.0.1:6071/health"

...
# Stop service.
./control stop
```

After the service started, you can view the running status of the service through logs. The address of the log file is ./var/app.log; if you need detailed logs, you can set the configuration item debug to true. You can view data about the internal status of the server through the debug script ./test/debug. For example, you can run bash ./test/debug to obtain the statistics about the internal status of the server.

##Configuration Instruction
The default configuration file is ./cfg.json. By default, the installation package includes an example configuration file cfg.example.json. The meanings of configuration items are described as follows.

```
{
    "debug": false, //true or false, indicates whether to enable the debug log.
    "http": {
        "enabled": true, //true or false, Indicates whether to enable this http port. This port is a control port and is used to send control commands, statistics commands, and debug commands to Graph.
        "listen": "0.0.0.0:6071" //Indicates the http port to be listened on.
    },
    "rpc": {
        "enabled": true, //true or false, indicates whether to enable this RPC port. This port is a data receiving port.
        "listen": "0.0.0.0:6070" //indicates the RPC port to be listened on.
    },
    "rrd": {
        "storage": "/home/work/data/6070" //Absolute path where the historical data file is stored. (If necessary, modify it to the appropriate path.)
    },
    "db": {
        "dsn": "root:@tcp(127.0.0.1:3306)/graph?loc=Local&parseTime=true", //Connection information of MySQL. By default, the user name is root, the password is empty, the host is 127.0.0.1, and the database is graph. (modify if necessary.)
        "maxIdle": 4  //Maximum number of connections allowed by the MySQL connection pool. You can keep the default value.
    },
    "callTimeout": 5000,  //Timeout for RPC, in ms
    "migrate": {  //Automatic migration of historical data during capacity expansion of Graph.
        "enabled": false,  //true or false, indicates whether Graph is in the data migration state.
        "concurrency": 2, //Number of concurrent connections during data migration. It is recommended to keep the default value.
        "replicas": 500, //Number of node copies needed by the consistency Hash algorithm. It is recommended to keep the default value. (It must be consistent with the configuration in Transfer.)
        "cluster": { //Old Graph instance list before capacity expansion.
            "graph-00" : "127.0.0.1:6070"
        }
    }
}
```

##Supplementary Note
After deploying the Graph component, modify the configurations of Transfer and Query so that the two components can address Graph.


