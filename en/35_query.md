##Query

The Query component provides the unified query entry for graphic data. It receives query requests, queries different metric data from corresponding Graph instances based on the consistency Hash algorithm, summarizes the obtained data, and finally returns the data to users.

##Source Code Deployment
```
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/query
go get ./...
./control build
./control pack
```


A tar.gz package will be packed at the last step. We can deploy with this package.

##Service Deployment

Service deployment includes configuration modification, starting service, verifying service, stopping service, etc. Before service deployment, you need to unzip the installation package to the deployment directory of the service.

```
# Modify configuration. Please refer to the following for meanings of configuration items. Note the configuration of the Graph cluster
mv cfg.example.json cfg.json
vim cfg.json

## Modify the configuration of the Graph cluster. By default, it is defined in ./graph_backends.txt.
vim graph_backends.txt


# Start service.
./control start

# Verify service. Here it is assumed that the service enables the 9966 http listening port. The verification result is ok, indicating that the service has started properly.
curl -s "127.0.0.1:9966/health"

...
# Stop service.
./control stop
```

After the service started, you can view the running status of the service through logs. The address of the log file is ./var/app.log. You can read graphic data through the query script ./scripts/query. For example, you can run bash ./scripts/query "ur.endpoint" "ur.counter" to query the graphic data corresponding to Endpoint="ur.endpoint" & Counter="ur.counter".

##Configuration Instruction

Note: Please ensure that the configurations of graph.replicas and graph.cluster are completely the same as those in Transfer.

```
{
    "debug": "false",   // indicates whether to enable the debug log.
    "http": {
        "enabled":  true,          // indicates whether to enable http.server.
        "listen":   "0.0.0.0:9966" // http.server listening address and port.
    },
    "graph": {
        "connTimeout": 1000, // Timeout for setting up a connection with the back-end Graph, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        "callTimeout": 5000, // Timeout for reading data from the back-end Graph, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        "maxConns": 32,      // Maximum number of connections, which is connection pool related configuration. It is recommended to keep the default value.
        "maxIdle": 32,       // Maximum number of idle connections, which is connection pool related configuration. It is recommended to keep the default value.
        "replicas": 500,     // Number of node copies needed by the consistency Hash algorithm. It should be consistent with the configuration in Transfer.
        "cluster": {         // Back-end Graph list, which should be consistent with the Transfer configuration; it is not supported to configure two addresses in one record.
            "graph-00": "test.hostname01:6070",
            "graph-01": "test.hostname02:6070"
        },
        "api": {  // API configuration needed to adapt to grafana.
            "query": "http://127.0.0.1:9966",     // http address of Query
            "dashboard": "http://127.0.0.1:8081", // http address of Dashboard
            "max": 500                            //maximum number of results returned by API
        }
    }
}
```
##Supplementary Note

After deploying the Query component, modify the configuration of the Dashboard component so that it can correctly address the Query component. Please ensure that the Graph list of the Query component is consistent with the Transfer configuration.
