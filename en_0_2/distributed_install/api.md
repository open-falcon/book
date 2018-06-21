<!-- toc -->

# API
Api component provides unified operation interfaces of restAPI. For example: Api component receives query request and query data of different Metrics in corresponding instance according to the consistent hashing algorithm. Finally, it summarizes the data and sends it back to user.

## Service Deployment
Service deployment includes changing configuration, starting the service, testing the service, stopping the service etc. Before this, you need to unzip installation package to deployment directory of the service.

```
# Change the configuration (the meaning of each setting is as follows) and pay attention to the configuration of Graph cluster
mv cfg.example.json cfg.json
vim cfg.json

# Start the service
./open-falcon start api

# Stop the service
./open-falcon stop api

# Check the log
./open-falcon monitor api

```

## Configuration Instruction

Attention: please make sure that the content of  `graphs` and the configuration of Transfer are **completely corresponding**

```
{
	"log_level": "debug",
	"db": {  // connection configuration related to database
		"faclon_portal": "root:@tcp(127.0.0.1:3306)/falcon_portal?charset=utf8&parseTime=True&loc=Local",
		"graph": "root:@tcp(127.0.0.1:3306)/graph?charset=utf8&parseTime=True&loc=Local",
		"uic": "root:@tcp(127.0.0.1:3306)/uic?charset=utf8&parseTime=True&loc=Local",
		"dashboard": "root:@tcp(127.0.0.1:3306)/dashboard?charset=utf8&parseTime=True&loc=Local",
		"alarms": "root:@tcp(127.0.0.1:3306)/alarms?charset=utf8&parseTime=True&loc=Local",
		"db_bug": true
	},
	"graphs": {  // deployment list of Graph module
		"cluster": {
			"graph-00": "127.0.0.1:6070"
		},
		"max_conns": 100,
		"max_idle": 100,
		"conn_timeout": 1000,
		"call_timeout": 5000,
		"numberOfReplicas": 500
	},
	"metric_list_file": "./api/data/metric",
	"web_port": ":8080",  // http monitor
	"access_control": true, // any user has administrator rights if it is set "false"
	"salt": "pleaseinputwhichyouareusingnow",  // salt value used in database password encryption
	"skip_auth": false, //user can visit api without authentication if it is set "true"
	"default_token": "default-token-used-in-server-side",  //for access authorization between modules of server
	"gen_doc": false,
	"gen_doc_path": "doc/module.html"
}



```

## Complementary Information
- After Api module is deployed, please change the configuration of dashboard module so that it can address Api module correctly
- Please make sure that the Graph list of api module and the configuration of Transfer are completely corresponding