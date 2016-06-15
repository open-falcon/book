##Alarm

##Design intention
The processing logic of the alarm event is not only sending mails and messages. To process event automatically, "alarm" should be able to call back the interfaces provided by users; sometimes there are so many alarm messages and mails that alarm merging is necessary for alarms with low priorities. And all this logic is finished in "alarm".

Alarm levels, such as P0/01/P2, etc., are configured when configuring alarm strategies. Each level of alarms is corresponding to a different redis queue. When "alarm" reads this data, we hope it reads P0 first, and followed by P1 and P5, because we hope to deal with the one with higher priority first. Hence, we will use the brpop command of redis.

##Source code installation

```
cd $GOPATH/src/github.com/open-falcon/alarm
go get ./...
./control build
./control pack

```

A tar.gz package will be packed at the last step. We can deploy with this package.

##Deployment instruction
"Alarm" is a single point. Unrecovered alarms should be stored in the memory of "alarm". "Alarm" needs to merge alarms, so only one instance can be deployed for "alarm", which should be improved in the future.

##Configuration instruction

The name of the configuration file must be cfg.json. We can change the configuration file based on cfg.example.json.

```
{
    "debug": true,
    "uicToken": "",
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:9912" # Unrecovered alarms can be viewed through the http page of "alarm"
    },
    "queue": {
        "sms": "/sms", # It should be configured as the same with "sender". Default setting is enough
        "mail": "/mail"
    },
    "redis": {
        "addr": "127.0.0.1:6379", # It should be the same redis address with "judge" and "sender"
        "maxIdle": 5,
        "highQueues": [
            "event:p0",
            "event:p1"
        ],
        "lowQueues": [
            "event:p2",
            "event:p3",
            "event:p4",
            "event:p5",
            "event:p6"
        ],
        "userSmsQueue": "/queue/user/sms", # It is OK to maintain default setting for these two queues
        "userMailQueue": "/queue/user/mail"
    },
    "api": {
        "portal": "http://falcon.example.com", # A portal address which can be visited through the Intranet
        "uic": "http://uic.example.com", # A uic (or fe) portal address which can be visited through the Intranet
        "links": "http://link.example.com" # A links address which can be visited through the Extranet
    }
}
```
The addresses of the portal and uic of the api part can be configured as an address available to visit through the Intranet, which can ensure high speed visit, while the address of links should be configured as an address available to visit through the Extranet. Pay attentions to this!

##Process management
We provide a control script to complete normal actions.
```
./control start	Start a process
./control stop	Stop a process
./control restart	Restart a process
./control status	View the process state
./control tail	Use the method of "tail -f" to view var/app.log
```
##Alarm merging
If a core service fails, massive alarms will be caused. To decrease the number of alarm messages, the alarm merging function is provided. This function writes alarm information into the "links" module, and the "links" module will return a url address to the "alarm" module, then the "alarm" module will send this url link to users. Therefore users only need to receive one message including a url address, and several alarms can be shown by clicking the url. 

Alarm merging will not be executed for the "event" queues configured in "highQueues", because those are high priority alarms and alarm merging is only for the events in "lowQueues". If you don't want to implement alarm merging to any event, you can allocate all the "event" queues into "highQueues". 

##Verification
If there is no error in the log located in the "var" directory, there should be no error. 

A web page showing "Unrecovered alarm list" will be shown by visiting the http address of the "alarm" module.

##Supplement

After the "alarm" module is set up, the "fe" configuration can be modified. You can configure shortcut:falconAlarm of the "fe" module into the http address of the "alarm", which can be visited through a browser.