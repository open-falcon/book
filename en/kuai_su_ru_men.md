##Getting Started

The system has been set. This section will show you how to use it step by step.

##Viewing monitor data

When agent is deployed in the machine and configured with heartbeat and transfer, the data will be collected automatically, and we can search and view monitor data in dashboard. Dashboard is a web project and visited through browsers. On the left side, we can type in endpoint. What is endpoint? What do we use to search for it? For the data collected by agent, endpoint is the name of machine. When ```hostname``` is performed in target machine, the output will be endpoint, then you can search with hostname. 

Then there is your search result. Okay, you select the previous check box, and click “check counter list”, counter subordinated to the endpoint will be listed. What is counter? ```counter=${metric}/sorted(${tags})```

For instance, we need to view cpu.idle, enter cpu in the counter search box and press enter key. There you can see cpu.idle, click, you can see a new page, and data of the latest one hour of the cpu.idle for the machine is shown in the chart. If you want to view the data of longer time period, there is a small triangle on the top right corner to extend menus and choose longer time period.

##picture

##How to configure alarm strategy

We have already learned how to view monitor data. If the data reach threshold value, for example cpu.idle is too small, how should we configure alarm?

##Configure alarm receiver

The alarm receiver of falcon is not a specific phone number or email address. Because phone numbers and email addresses are easily changed, it is very troublesome to modify every related configuration when they are changed. We maintain the user contact information in a system called UIC (Go version of UIC, i.e. Falcon-fe project is recommended for new users). If we need to modify their phone numbers or email addresses, we only need to modify once in UIC. Alarm receiver is not a single person, but a group (called Team in UIC). For instance, any component in the falcon system goes wrong, an alarm should be sent to falcon team, including operation, maintenance and development people of falcon. In this way, new employees only need to be added to the Team, and once an employee leaves the company, it only needs to be deleted from the Team of falcon.

UIC is visited through browsers. If LDAP is enabled, LDAP account should be used for log-in, if not, then the user can register one or ask administrator to help register. I now created a Team named falcon, added myself for testing later on.

##picture

##Create HostGroup

For example, we need to monitor the port of the falcon-judge component. First, we create a HostGroup, and add in all machines configured with the falcon-judge module. We can add or delete machines in the HostGroup directly when the machines need to expand or log off, the alarm strategy will take or lose effect. We name the HostGroup as: sa.dev.falcon.judge. The name has multiple meanings. Sa is our department. Dev is our team. Falcon is the project name. Judge is the component name. There is a lot of information in the name. This naming method is recommended for everyone for it is easy to manage.

##picture

If an error is reported when adding machine to the group, the host list in the portal database should be checked to see if there is any related machine in it. Then where do machines in host list come from? There is a configuration for agent heartbeat(hbs), and heartbeats will be sent to hbs by agent to provide its information such as ip, hostname, agent version, etc. to hbs, and hbs will write all these information into the host list. If there is no data in the host list, we need to check if the link is unobstructed. 

##Create strategy template

There is a Templates link in the top of portal, which is the entrance for strategy template management. We enter and create a template named sa.dev.falcon.judge, the same name with HostGroup, and configure a port monitor inside. Generally, there are two methods of process monitoring. One is to see if the process itself is alive, and the other is to see if the port is monitoring. Here we use port monitoring.

##picture

The add button in the top right corner is for adding strategy. One template can have multiple strategies. Here we only add one. Then we can configure alarm receiver as falcon which is the Team created in UIC previously. 


##Binding HostGroup with templates

One template can be bound to multiple HostGroups. Now let us go back to the page of HostGroups again to find the HostGroup called sa.dev.falcon.judge. There are several hyperlinks on the right side, click [templates] to enter a new page, and type in the name of the template and bind. 

##picture

##Supplement

The above steps are completed and the configuration is also completed. If judge component crashed and the port is not monitoring, there will be an alarm. But we should not crash the judge component on purpose for testing the alarm is functioning. Because judge itself is for judging alarms, we cannot judge if we crash it. Falcon is not perfect at present, and it cannot be used for monitoring its own components. For testing, we can change the strategy configuration of port monitor into a port which is not monitoring, and then we can trigger an alarm.

The above strategies are only for port monitoring of falcon-judge. Then if we need to add some load monitoring for all machines in falcon project, what shall we do? 

1.Create a HostGroup: sa.dev.falcon, add all falcon machines in it

2.Create a template: sa.dev.falcon.common, add some strategies like cpu.idle, load.1min, etc.

3.Bind sa.dev.falcon.common with the HostGroup of sa.dev.falcon

Attached: Configuration example of sa.dev.falcon.common

##picture

You may not know the name of each index. However, one must know the metrics of the data pushed by you. The data of agent push can refer to: https://github.com/open-falcon/agent/tree/master/funcs

##How to configure strategy expression

Strategy expression, i.e. Expression, for details, you can refer to HostGroup and Tags Design Concept, and below is only an example: 

##picture

The configuration of above example means: all the instances of the falcon-judge module, if qps is above 1000 for over 3 consecutive times, an alarm should be reported to falcon alarm group. 

Expression does not need to be bound with HostGroup, enjoy it