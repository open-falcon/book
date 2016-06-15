##Agent

Agent is used to collect computer load monitoring indicators, such as cpu.idle, load.1min, disk.io.util, etc. and push them to Transfer every 60 seconds. Agent sets up a persistent connection with Transfer to enable fast data transfer speed. Agent provides an http interface /v1/push to receive some data pushed manually by users and sends the data to Transfer at a fast speed through the persistent connection.


##Source Code Installation

```
cd $GOPATH/src/github.com/open-falcon/agent
go get ./...
./control build
./control pack
```
A tar.gz package will be packed at the last step. We can deploy with this package.

##Deployment Instruction
Agent needs to be deployed on all the computers to be monitored. For example, if a company has 100,000 computers, 100,000 agents need to be deployed. Agent consumes few resources, so there is no need to worry about this.

##Configuration Instruction
The name of the configuration file must be cfg.json. We can change the configuration file based on cfg.example.json.
```
{
    "debug": true, # Controls the output of some debug information. It is usually set to false in the production environment.
    "hostname": "", # endpoint is set as hostname when Agent collects data and sends it to Transfer. By default, endpoint is obtained through "hostname". If hostname is configured in the configuration file, the configured hostname is used.
    "ip": "", # Agent sends its own IP address to hbs when keeping heartbeat communication with hbs. Agent automatically detects the IP address of the local computer. If you do not want Agent to detect the IP address automatically, you can modify the configuration manually.
    "plugin": {
        "enabled": false, # By default, the plugin mechanism is not enabled.
        "dir": "./plugin", # Clones git repo where plugin scripts reside to this directory.
        "git": "https://github.com/open-falcon/plugin.git", # it repo address where plugin scripts reside.
        "logs": "./logs" # Logs for plugin execution. If errors occur during plugin execution, you can go to this directory to check logs.
    },
    "heartbeat": {
        "enabled": true, # Here, enabled needs to be set to true.
        "addr": "127.0.0.1:6030", # Address of hbs, and the port is the RPC port of hbs.
        "interval": 60, # Heartbeat period, in seconds.
        "timeout": Timeout for connection with hbs, in milliseconds.
    },
    "transfer": {
        "enabled": true, # Here, enabled needs to be set to true.
        "addrs": [
            "127.0.0.1:8433",
            "127.0.0.1:8433"
        ], # Address of Transfer, and the port is the RPC port of Transfer. Multiple Transfer addresses are supported and Agent will ensure HA.
        "interval": 60, # Collection period, in seconds. It means that Agent collects and sends data to Transfer every minute.r
        "timeout": 1000 # Timeout for connection with Transfer, in milliseconds.
    },
    "http": {
        "enabled": true, # Indicates whether to listen on an http port.
        "listen": ":1988" # Address of the listening port if listening is implemented.
    },
    "collector": {
        "ifacePrefix": ["eth", "em"] # By default, traffic only of the network cards with the name prefix of eth or em will be collected. If it is left empty, traffic of all network cards including lo will be collected. Traffic information of each network card can be viewed from /proc/net/dev.
    },
    "ignore": { # By default, over 200 metrics are collected. You can set Agent not to collect metrics by ignore.
        "cpu.busy": true,
        "mem.swapfree": true
    }
}
```

##Process Management

We provide a control script to complete common operations.

```
./control start Start the process
./control stop Stop the process
./control restart Restart the process
./control status View process status
./control tail View var/app.log using tail -f
```

##Verification

Check whether the log in the var directory is normal, or use the browser to visit the 1988 port. In addition, Agent provides a --check parameter, which can check whether Agent runs properly on the current computer.

```
./falcon-agent --check
```

##/v1/push Interface

By our original intention of design, users are not expected to connect directly to Transfer for sending data but to forward data through the /v1/push interface of Agent. Interface usage example:

```
ts=`date +%s`; curl -X POST -d "[{\"metric\": \"metric.demo\", \"endpoint\": \"qd-open-falcon-judge01.hd\", \"timestamp\": $ts,\"step\": 60,\"value\": 9,\"counterType\": \"GAUGE\",\"tags\": \"project=falcon,module=judge\"}]" http://127.0.0.1:1988/v1/push
```

##Video Course

We recorded a video for this module to provide interpretation at the source-code level:
http://www.jikexueyuan.com/course/2242.html





