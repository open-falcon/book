<!-- toc -->

# Agent

Agent is used for collecting data of machine load, like cpu.idle, load.1min, disk.io.util and so on. The data is pushed to Transfer every 60 seconds. A persistent connection is set up between Agent and Transfer, so the data is sent at higher speed. Agent provides an http port "/v1/push" for receiving data pushed by user manually, and quickly forwards these data to Transfer via the persistent connection.

## Additional Information of Agent Deployment

Agent needs to be deployed at all monitored machines. For example, if there are 100,000 machines in our company, then 100,000 Agents needs to be deployed. That number is not insane because Agent itself costs little resource. 

## Configuration Instructions

The name of configuration file must be "cfg.json" and it can be changed based on "cfg.example.json".

```
{
    "debug": true,  # used for controlling the information export of debug; it is usually set as "false" in production environments
    "hostname": "", # Agent collects data and send them to Transfer and then "endpoint" becomes the hostname from "hostname"; if hostname is set in settings then that hostname is used
    "ip": "", # Agent and hbs will send their ip address to hbs when they heartbeat and agent will detect the ip address of this computer; if you do not want the detection to start, you can modify the setting manually 
    "plugin": {
        "enabled": false, # plug-in mechanism is off by default
        "dir": "./plugin",  # copy the git repo where the scrpits of plugin are saved to this directory
        "git": "https://github.com/open-falcon/plugin.git", # git repo address where the scrpits of plugin are saved
        "logs": "./logs" # log of plugin execution; if there is a problem during plugin execution, you can check the log here
    },
    "heartbeat": {
        "enabled": true,  # enabled here must be set to "true"
        "addr": "127.0.0.1:6030", # address of hbs and the port belongs to rpc of hbs
        "interval": 60, # heartbeat time interval, measured in second
        "timeout": 1000 # time-out setting for hbs connection, measured in millisecond
    },
    "transfer": {
        "enabled": true, 
        "addrs": [
            "127.0.0.1:18433"
        ],  # the address of Transfer; and the port belongs to rpc of Transfer; more than one Transfer addresses are supported here because Agent will guarantee HA
        "interval": 60, # collecting interval measured in second, which means Agent collects data and send them to Transfer every minute
        "timeout": 1000 # time-out setting for Transfer connection, measured in millisecond
    },
    "http": {
        "enabled": true,  # whether to monitor http port
        "listen": ":1988",
        "backdoor": false
    },
    "collector": {
        "ifacePrefix": ["eth", "em"], # the default setting will only collect the information of data traffic of netcards whose name prefix is "eth" and "em"; if the setting is null, then it will collect the information of all netcards including netcards with "lo" prefix; the information of data traffic will displayed at "/proc/net/dev"
        "mountPoint": []
    },
    "default_tags": {
    },
    "ignore": {  # over 200 Metrics are collected; "ignore" setting will shut down this collection
        "cpu.busy": true,
        "df.bytes.free": true,
        "df.bytes.total": true,
        "df.bytes.used": true,
        "df.bytes.used.percent": true,
        "df.inodes.total": true,
        "df.inodes.free": true,
        "df.inodes.used": true,
        "df.inodes.used.percent": true,
        "mem.memtotal": true,
        "mem.memused": true,
        "mem.memused.percent": true,
        "mem.memfree": true,
        "mem.swaptotal": true,
        "mem.swapused": true,
        "mem.swapfree": true
    }
}
```

## Process Management

```
./open-falcon start agent  启动进程
./open-falcon stop agent  停止进程
./open-falcon monitor agent  查看日志

```

## Verification

Check the log at var directory to see if it is normal， or visit its 1988 port using the browser. Besides, Agent provides a `--check` parameter which can check if Agent can run normally in current machine

```
./falcon-agent --check
```

## Port /v1/push

Transfer is not designed for sending data from users. We actually hope users will forward the data via Agent's port "/v1/push". Here is an example of how it is used:

```
ts=`date +%s`; curl -X POST -d "[{\"metric\": \"metric.demo\", \"endpoint\": \"qd-open-falcon-judge01.hd\", \"timestamp\": $ts,\"step\": 60,\"value\": 9,\"counterType\": \"GAUGE\",\"tags\": \"project=falcon,module=judge\"}]" http://127.0.0.1:1988/v1/push
```

## Video Tutorial

We recorded a video tutorial for this module on source-code-level: http://www.jikexueyuan.com/course/2242.html
