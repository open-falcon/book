# 5.3 Plugin Mechanism

As for Plugin mechanism, we must highlight the following contents before we get right to the point:

Plugin could be seen as an extension of agent. As for monitoring collection of business systems, it is better to put collection script into business application release package rather than using plugin. So keeping pace with business code’s going online and upgrading, it is easier to manage.

To use Plugin, the steps are as follows:

##1. Writing collection script
It doesn’t matter about which language you use, provided that there is runtime environment in the target host. The script itself should have execute permission. The data that was collected can be directly printed to stdout, then agent can intercept and push the data to server. Data format is json, for example:
```
[root@host01:/path/to/plugins/plugin/sys/ntp]#./600_ntp.py
[{"endpoint": "host01", "tags": "", "timestamp": 1431349763, "metric": "sys.ntp.offset", "value": 0.73699999999999999, "counterType": "GAUGE", "step": 600}]
```
Note, this json data is a list.

##2. Uploading the script to git

Plugin script is also code, so you’d better use git or svn to manage it. We use git to manage here. If gitlab is not set up within the company, you can use gitcafe or coding.net, and push the written script to git store. , For example, we place 600_ntp.py mentioned above to sys/ntp catalogue in the git store. Note that you should add execute permission to this script before being pushed to git store.

##3. Checking agent configuration
When you deployed agent before, you must have noticed that plugin was configured in the agent configuration file. Now you can use it to configure git store address, setting enabled to true. Note that configured git store address can be pulled from any hosts, namely starting with git:// or https:// . If agent have been deployed to all hosts in the company before, it is a little hard to change configuration manually. As we have discussed it previously, we use ops-updater to manage it.

##4. Pull plugin script

Because agent opened a http port: 1988, we can curl the following addresses one by one: http://ip:1988/plugin/update. This can allow agent to git pull plugin store. Why don’t we make it regularly to pull this store? It will put too much pressure on git server. Take it easy, don’t pull it to failure.

##5. Let plugin run

At the last step, we pulled plugin script to all hosts, but plugin didn’t execute. It is configured on the portal as to which host executes which plugin script. Actually I want to do it in this way: once plugin is pulled, the plugin executes instantly. But in practice, because some plugin could not still run on all hosts, it is managed via configuration control on the portal. Find HostGroup that needs executing plugin on the portal, and click the corresponding plugin hyperlink. As for 600_ntp.py under the sys/ntp catalogue in the above example, just directly bind sys/ntp to it. All plugins under sys/ntp will be executed automatically.

##6. Supplement content
After configuration on the portal was finished, it would not take effect right away. Because a synchronization process is required, agent can be accessed through calling hbs’s port during 1 to 2 minutes. In the above example, we have bound with sys/ntp which is actually a catalogue. All plugins under the catalogue will be executed. What kind of files will be considered as plugin? The file names starting with a number + underline (_): the number represents step, which is how long it would run every time, specified in second. For example, a name of 60_a.py notifies agent that the plugin runs once every 60 seconds. Sub catalogs and other files with different naming types under sys/ntp catalogue will be ignored.