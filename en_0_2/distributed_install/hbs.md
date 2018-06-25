<!-- toc -->

# HBS(Heartbeat Server)

All agents of our company will be connected to HBS which will send an HBS request every minute. 

## Design Intention

There is an Host list in the database of Portal that maintains all the information of the company's machines, like hostname and ip. The data in this list is usually synchronized from the company's CMDB. But some small companies do not have a CMDB, and they have to enter data manually in Host list, which is exhausting. Therefore, we added the first feature to HBS: Agent will send the information, like hostname, ip, agent version, plugin version etc., while it is sending heartbeat information to HBS and HBS will refresh Host list.

The biggest feature of Falcon-agent is that it is capable of self discovering. It will collect information without configuration. That of CPU, memory, disk, data of network card traffic will be collected automatically. Besides collecting basic information, we also need to monitor the port viability and the number of process. Then shall we automatically collect the monitor ports and the numbers of process? We think the answer is no. Because the amount of the data is huge. Also, it is a waste of time because most users do not care about these data. So we made another change. We only collect what user configures. For example, if a user configures the monitor of port 80 of one machine, we will collect the viability the port 80 of this machine. How can Agent know which port and process it needs to collect? It can ask HBS for details. HBS will read the database of Portal and send the data back to Agent.

Afterwards, we will introduce a component for judging alarms: Judge. Judge requires all the alarm policies. It is not sensible to let Judge read the database of Portal, because the amount of Judge instance is huge. If a company has hundreds of thousands of machines， the amount of Judge instance may reach several million. It is a pressure to let millions of Judge instance visit the database of Portal. Since it is inevitable to let HBS visit the database of Portal, the best solution is to let HBS pick up all the alarm policies and save them in memory, which will be requested by Judge. The pressure of the database of Protal is significantly reduced.


## Deployment Information

HBS supports horizontal scaling. At least two instances need to be deployed to guarantee it is available. Usually, one instance can deal with 5,000 machines. That is to say, if a company has 100,000 machines, it is enough to deploy 20 HBS instances. If lvs is erected before, you just need to configure lvs vip in Agent.

## Deployment Instruction

The name of configuration file must be "cfg.json" and you can change it basing on "cfg.example.json".

```
{
    "debug": true,
    "database": "root:password@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true", # database address of Portal
    "hosts": "", # there is a Host list in the database of Portal; if the list is synchronized from other system then the setting should be sync or stay null by default
    "maxIdle": 100,
    "listen": ":6030", # RPC address that HBS monitors
    "trustable": [""],
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6031" # http address that HBS monitors
    }
}
```

## Process Management

```
# Start the service
./open-falcon start hbs

# Stop the service
./open-falcon stop hbs

# Check the log
./open-falcon monitor hbs 

```

## Complementary Information

If you deploy Agent before HBS, then you need to change the setting of Agent after deploying HBS: set the value of "enable" setting in Heartbeat of Agent configuration to "true", and set the value of "hddr" to the rpc address of HBS. If the configuration file remains default, rpc port is 6030 and http port is 6031. Pay attention that the Rpc port of HBS is set in Agent configuration.


## Video Tutorial

We recorded a video tutorial for HBS component at source-code level: http://www.jikexueyuan.com/course/1873.html

