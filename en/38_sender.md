##Sender

In the last section, we use http interface specification to shield the email and message-sending problem. The Sender module is used to invoke the email and message-sending interface provided by each company.

##Design Intention

Each company provides its email and message-sending interface. It is not suitable to call these interfaces to send alarms immediately after alarms are generated because these interfaces may be unable to process huge concurrent calls and may have slow processing speed. This will slow down our processing logic. Therefore, it is a better way to make email and message sending asynchronous. 

We provide a message Redis queue and an email Redis queue. When a message needs to be sent, the content of the message is written into the message Redis queue. When an email needs to be sent, the content of the email is written into the email Redis queue. One worker thread pool with preset size processes each queue.

With the buffer of queues, even if a large number of alarms is generated at a moment to bring burst traffic of email and message sending, it will not cause great impact on the email and message sending interfaces.

##Source Code Installation

```
cd $GOPATH/src/github.com/open-falcon/sender
go get ./...
./control build
./control pack
```
A tar.gz package will be packed at the last step. We can deploy with this package.


##Deployment Instruction

The Sender module and Redis queues can be deployed on the same computer. One Sender is sufficient even if a company has hundreds of thousands of computers.

##Configuration Instruction

The name of the configuration file must be cfg.json. We can change the configuration file based on cfg.example.json.

```
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6066"
    },
    "redis": {
        "addr": "127.0.0.1:6379", # The Redis address configured here needs to be the same as that configured in Judge and Alarm later.
        "maxIdle": 5
    },
    "queue": {
        "sms": "/sms", # name of the message queue. It is recommended to keep the default value. There will be a same configuration in Alarm.
        "mail": "/mail" # name of the email queue. It is recommended to keep the default value. There will be a same configuration in Alarm.
    },
    "worker": {
        "sms": 10, # maximum number of concurrent calls for the message interface
        "mail": 50 # maximum number of concurrent calls for the email interface
    },
    "api": {
        "sms": "http://11.11.11.11:8000/sms", # message sending interface provided by each company. The IP address 11.11.11.11 is only an example.
        "mail": "http://11.11.11.11:9000/mail" # email sending interface provided by each company.
    }
}
```
If no email sending interface is provided, Open-Falcon mail-provider can be used.

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

The http listening port is configured in the configuration file of Sender. We can access the /health interface to check whether ok is returned. All of our Go back-end modules provide the /health interface. You can verify the above configuration as follows:

```curl 127.0.0.1:6066/health```

In addition, you can view the log of Sender in the var directory.

##Video Course

We recorded a video for the Sender module to provide interpretation at the source-code level:
http://www.jikexueyuan.com/course/1641.html

