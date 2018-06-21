<!-- toc -->

# Judge

Judge is designed for judging the alarm. Agent sends data to Transfer and Transfer will not only forward them to Graph component for drawing diagrams but also to Judge for judging whether the alarm goes off.

## Design Intention

Since it's a huge of monitor data, processing the data is far beyond the capability of a single machine. So data partition strategy is necessary. Transfer finishes the partition using the consistent hashing algorithm and each Judge only needs to process a small segment of data. So the feature of judging alarm can't be built in the data receiving part, Transfer, but the component after Transfer. 


## Deployment Instruction

Judge monitors an http port while providing another http port "/count". You will know how many data the current Judge instance has processed when you visit this port. We recommend that one Judge instance processes 500,000 to 1,000,000 pieces of data, taking up 5 to 10 GBs of memory. If the memory of the machine you use is relatively big, like 128GBs, more than one Judge instances can be deployed on this machine. 

## Configuration Information

The configuration file must be named "cfg.json" and it can be changed based on "cfg.example.json".

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
        "servers": ["127.0.0.1:6030"], # hbs最好放到lvs vip后面，所以此处最好配置为vip:port
        "timeout": 300,
        "interval": 60
    },
    "alarm": {
        "enabled": true,
        "minInterval": 300, # the minimum time interval between to consecutive alarms measured in second, the value of which can remain as default
        "queuePattern": "event:p%v",
        "redis": {
            "dsn": "127.0.0.1:6379", # use the same redis of alarm and sender
            "maxIdle": 5,
            "connTimeout": 5000,
            "readTimeout": 5000,
            "writeTimeout": 5000
        }
    }
}
```

Further detail of item "remain":
Remain defines the amount of certain data saved in Judge's memory. An example is how many values of "cpu.idle" of host01 in saved in memory. As for alarm configuration, the number after "#", like "all(#3)", cannot be larger than the number minus one in the "remain" item. Usually, we will leave it as default. 

## Process Management

We provide a control script to perform common operation

```
# Start the service
./open-falcon start judge

# Stop the service
./open-falcon stop judge

#  Check the log
./open-falcon monitor judge 
```

## Video Tutorial

We recorded a video tutorial for Judge component at source-code level: http://www.jikexueyuan.com/course/1850.html

