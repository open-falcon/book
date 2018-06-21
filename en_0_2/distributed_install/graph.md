<!-- toc -->

# Graph

Graph is a component used for storing data of graph. Graph component receives monitor data sent from Transfer component, processes query request of Api component and returns data of graph. 

# Service Deployment
Service deployment includes changing configuration,, starting the service, testing the service, stopping the service etc. Before this, you need to unzip the the installation package to deployment directory of the service.

```
# Change the configuration (the meaning of each setting is as follow)
mv cfg.example.json cfg.json
vim cfg.json

# Start the service
./open-falcon start graph

# Stop the service
./open-falcon stop graph

# Check the log
./open-falcon monitor graph

```

## Deployment Instruction
The configuration file is "./cfg.json" and there is an example configuration file example in installation package by default. The meaning of each setting is as follows.

```
{
    "debug": false, //whether to enable debug log or not
    "http": {
        "enabled": true, //whether to open this http port or not which is used for sending control command , statistical command and debug command to Graph
        "listen": "0.0.0.0:6071" //refer to http monitor port
    },
    "rpc": {
        "enabled": true, //whether to open this rpc port or not which is used for receiving data
        "listen": "0.0.0.0:6070" //refer to rpc monitor port
    },
    "rrd": {
        "storage": "./data/6070" // address where history data are saved（change if it is necessary）
    },
    "db": {
        "dsn": "root:@tcp(127.0.0.1:3306)/graph?loc=Local&parseTime=true", //connection information of MySQL: default username is root，password is nuul，host is 127.0.0.1 and database is graph(change if it is necessary)
        "maxIdle": 4  //connection pool configuration of MySQL and the number defines the maximum number of connections; it can stay as the default value
    },
    "callTimeout": 5000,  //time-out setting of calling RPC, measured in millisecond
    "migrate": {  //history data will migrate when Graph is in expansion
        "enabled": false,  //show whether Graph is in data migration or not
        "concurrency": 2, //number of concurrent connections in data migration; kept as default value on recommendation
        "replicas": 500, //number of node replicas that the consistent hashing algorithm requires; kept as default value on recommendation (must be identical with the one in Transfer configuration)
        "cluster": { //old list of Graph instance before expansion
            "graph-00" : "127.0.0.1:6070"
        }
    }
}

```

## Complementary Information
After Graph component is deployed, please change the configuration of Transfer and Api so that they can address Graph correctly.
