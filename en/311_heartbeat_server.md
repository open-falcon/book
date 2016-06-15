##HBS (Heartbeat Server)

The Heartbeat Server, which all the agents in the company are connected to, sends one heartbeat request once a minute. 

##Design intention

There is a host table in the Portal database maintaining the information of all the company's machines, e.g. hostname, ip, etc. Usually, the data in the table is acquired by synchronizing with the company's CMDB. But some small companies don't have CMDB, and data needs to be input to the host table manually, which is quite inconvenient. So we set HBS a function: when the agent sends heartbeat information to HBS, information of hostname, ip, agent version, plugin version and others will also be sent to HBS. And HBS is responsible for updating the host table.

Falcon-agent has an obvious characteristic: self-discovering. This means there is no need to configure which data to collect, it will collect data automatically. Information such as CPU, memory, disk, network card traffic will be collected automatically. We need not only to collect the basic information, but also monitor the port and the number of processes. Should we also collect the monitored ports and the number of each process automatically? No, because the data quantity is very large, which is resource-consuming, and most users don't care about this information.
Therefore, we change the collection mode, and only collect the information configured by users. For example, only if users configure that the port 80 of a computer should be monitored, should we collect the survivability of the computer's port 80. Then how the agent knows which ports and processes to collect? The agent will ask the information from HBS: HBS will read the database of Portal and return it to the agent.

Now we introduce a component to judge alarms: Judge. Judge needs to get all alarm strategies. Should we make Judge read the database of Portal? This method is not so good. Because the number of instances of Judge is very large, if the company has hundreds of thousands of machines, then the number of instances of Judge will be hundreds. And there will be hundreds of Judge instances to visit the Portal database, which can bring great pressure to the network. Since HBS will visit the database of Portal at all events, we can make HBS to get all alarm strategies and cache them in the memory, then Judge can request from HBS. As thus, the pressure of the Portal database will be decreased greatly.

##Source code installation

```
cd $GOPATH/src/github.com/open-falcon/hbs
go get ./...
./control build
./control pack
```
A tar.gz package will be packed at the last step. We can deploy with this package.

##Deployment instruction

HBS can be scaled horizontally. We should at least deploy two instances to ensure the availability. Generally, one instance can deal with 5,000 machines, so if a company has 100,000 machines, we need only deploy 20 HBS instances, set up lvs in the front and configure lvs vip in the agent.

##Configuration instruction

The name of the configuration file must be cfg.json. We can change the configuration file based on cfg.example.json.

```
{
    "debug": true,
    "database": "root:password@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true", # Address of Portal's database
    "hosts": "", #There is a host table in the portal database, if the data in the table is synchronized from other systems, this should be configured as "sync", or the default setting should be maintained and this place should be left empty.
    "maxIdle": 100,
    "listen": ":6030", # the rpc address monitored by HBS
    "trustable": [""],
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6031" # the http address monitored by HBS
    }
}
```
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
Visit the /health interface to verify whether HBS works normally.

```curl 127.0.0.1:6031/health```

The other method is to view the log of HBS which is under the var directory.

##Supplement
If you deploy the agent firstly and HBS after, we should return to change the deployment of the agent after finishing HBS deployment: enable the heartbeat part of the agent as true, and set the address as the rpc address of HBS. If the configuration file of HBS remains default, the rpc port should be 6030, and the http port should be 6031. The rpc port of HBS should be configured in the agent. Be careful to avoid mistakes.

##Video course

We recorded a video for the HBS module to provide interpretation at source code level: http://www.jikexueyuan.com/course/1873.html
