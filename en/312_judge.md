##Judge

Judge is used for alarm judgment. The agent pushes data to Transfer, then Transfer will not only transfer the data to the Graph component for drawing, but also transfer it to Judge to judge whether an alarm should be triggered.

##Design intention
Because there is large data quantity in the monitoring system, which obviously one machine is not enough to deal with, so a data fragment scheme is necessary. Transfer fragments data through consistent hashing, and then every Judge only needs to deal with a fraction of data. Therefore, the function to judge alarms should not be located in the direct data receiving end: Transfer, but in the module behind Transfer.

##Source code installation
```
cd $GOPATH/src/github.com/open-falcon/judge
go get ./...
./control build
./control pack
```

A tar.gz package will be packed at the last step. We can deploy with this package.

##Deployment instruction
Judge monitors a http port and provide a http interface: /count, by visiting which we can know the data quantity that the current Judge instance is dealing with. A recommended method is that a Judge instance deals with 500,000 - 1,000,000 pieces of data with a 5G - 10G memory. If the memory of the used physical machine is large, e.g. 128G, several Judge instances can be deployed in one physical machine.

##Configuration instruction

The name of the configuration file must be cfg.json. We can change the configuration file based on cfg.example.json.

```
{
    "debug": true,
    "debugHost": "nil",
    "remain": 11,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6081"
    },
    "rpc": {
        "enabled": true,
        "listen": "0.0.0.0:6080"
    },
    "hbs": {
        "servers": ["127.0.0.1:6030"], # It should better be located behind lvs vip, so it is recommended to configure this place as vip:port
        "timeout": 300,
        "interval": 60
    },
    "alarm": {
        "enabled": true,
        "minInterval": 300, # The fewest seconds between two continuous alarms. Default setting should be maintained.
        "queuePattern": "event:p%v",
        "redis": {
            "dsn": "127.0.0.1:6379", #  Uses the same redis with "alarm" and "sender"
            "maxIdle": 5,
            "connTimeout": 5000,
            "readTimeout": 5000,
            "writeTimeout": 5000
        }
    }
}
```
Let's explain the "remain" configuration in detail:  "remain" specifies how many points should be saved for one data in the judge memory, for example, how many cpu.idle values of the host01 machine can be saved in the memory at most. When configuring alarms, e.g. all (#3), the number behind # should not exceed "remain-1". Usually maintaining default setting is enough.

##Process management
We provide a control script to complete normal actions.
```
./control start	Start a process
./control stop	Stop a process
./control restart	Restart a process
./control status	View the process state
./control tail	Use the method of "tail -f" to view var/app.log
```
##Verification
Visit the /health interface to verify whether Judge works normally.

```curl 127.0.0.1:6081/health```

The other method is to view the log of Judge which is under the var directory.

##Video course

We recoded a video for the Judge module to provide interpretation at source code level: http://www.jikexueyuan.com/course/1850.html