<!-- toc -->

Before the instruction of plugin mechanism I must explain something.

> Plugin can be considered as a functional complement of Agent. We advise users to put the script of data collection in the pack of the business program. It is not wise to code the data collection as a plugin. The script becomes online , offline or expanded as the service code does, which is easier to manage.

Here are the steps for using plugin:

**1. Write the script of data collection**

It does not matter what language the script is written in as long as the target machine has the operation environment and the script itself has executable permissions. Just print the data to stdout after they are collected. Agent will intercept the data and push them to the server in jason format. Here is an example.

```bash
[root@host01:/path/to/plugins/plugin/sys/ntp]#./600_ntp.py
[{"endpoint": "host01", "tags": "", "timestamp": 1431349763, "metric": "sys.ntp.offset", "value": 0.73699999999999999, "counterType": "GAUGE", "step": 600}]
```

Notice: This piece of json data is a list.

**2. Upload the script to git**

Since the script the plugin is also a type of code, it is better to manage them throught git or svn. We choose git here. If the company does not have gitlab in the inner network, it can be replaced by Gitcafe or coding.net. Push the finished script to the git lab with executable permissions, like the 600_ntp.py above, and save it in the sys/ntp directory.

**3. Check the configuration of Agent**

I believe that everybody has noticed that a plugin is configured when you deploy Agent. Now it is time to use it. Configure the address of git lab and set the "enabled" to true. The address of git lab must start with `git://` or `https://`, so it can be pulled by any machine. If the Agent has been deployed in every machine of the company, it can be a Little troublesome to change the configuration manually. We have talked about this situation that this is the best time to use the [ops-updater](https://github.com/open-falcon/ops-updater).

**4. Pull the plugin script**

Agent opens an http port 1988. We can curl the address http://ip:1988/plugin/update one by one. This will let Agent proactively pull the git lab. The reason we did not design this process as a regular action is because we do not want to add to much pressure on the server of git or it will crush.

**5. Get Plugin Executed**

We have pulled the script of plugin to all the machine, but it is still not executed. Which machine executes what script needs to be configured on portal. Actually my origial design is that the plugins will be executed as long as they are pulled, which I really want to make it happen. But in real experiment, some plugins are  not executable in all machines. Due to this situation, I have to leave the plugin execution setting in the configuration. Users need to find the hostgroup of the plugin that they want to execute in portal, and then open the corresponding link. For 600_ntp.py in the directory of sys/ntp, just bind sys/ntp to it and all the plugins in the directory of sys/ntp are executable.

**6. Complemantary Information**

The configuration on portal will not be valid until it is synchronized. Agent picks it up by invoking the port of hbs, which will take one or two minutes. In the example about we bound sys/ntp that is a directory, and all the plugins in this directory will be executed. But what kind of file will be considered as plugin? The answer is the one whose name is made of numbers and begins with "~". The numbers stand for step, which means the execution time internal measured in second. For example, the name "60_a.py" tells agent to execute this plugin every 60 seconds. Sub-directories of sys/ntp and other naming patterns will be neglected.