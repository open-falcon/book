# 9.5 QQ Group Q&A

##Q&A from qq Goup (373249123)

##Alarm

Q: Can we send an alarm email to find Top 3 programs occupy traffic, when network traffic exceeds certain threshold value? Can we write like this: each (endpoint=xxx metric=yyy tag1=tttt) expression? @ Nie An_Xiao Mi

A: Yes. If endpoint is not the name of the machine, each expression can be used for configuring alarm strategy. If endpoint is the name of the host, the way of hostgrp + template is suggested to use for configuring strategy - this way is easier for management. (Of course, if endpoint is not the name of the host, the way of hostgrp + template can also be used, only need to insert endpoint in hostgrp)

Q: By the way, when the maximum alarm times is 3, where can we set up the time internals between each alarms for the first 3 times

A: For example, your data comes up once a minute, theoretically it only alarms three times respectively at the third minute, sixth minute, ninth minute, and then no longer alarms. But in this way we think it is too frequent, therefore, there is a minimum alarm setup in judge, the default is 5 minutes, that is to say: there has to be at least 5 minutes between two alarms: the third minute, the eighth minute, the thirteenth minute. Link does not have alarm combination, Alarm only combines alarms of the same type within one minute.  

Q: How do I know my strategy is synchronized successfully? 


A: curl -s "judge-hostname:port/strategy/host.test.01/cpu.idle", this is for checking the corresponding strategy of host host.test.01 & metric for cpu.idle. 

Q: I want to determine the alarm occurs when disk space is less than 10% and free space is less than 1TB, for some storage with high capacity, their free space for 10% might still have several TB. Can we realise that?

A: Currently our framework does not support combined strategy.

Q: Now I made a ping test, it will ping through two nodes A, B, correspond to 2 tags, ping_server=A, ping_server=b. I want to make an alarm item, when both node A, B cannot connect to the target host, I will trigger alarm. There is another example, I want to set up alarm, when connecting numbers of nginx wave more than 20% and connecting numbers are more than 1000, the alarm will be triggered. Perhaps the number of nginx for some machine business is very small, it is already 100% when waves from connecting number 1 to connecting number 2.

A: This can be the situation of group monitoring, it will alarm only when both two machines shut down in the group, and will not alarm when one of them shut down. If it is for two different collection items, combination strategy cannot be realised based on current framework, because two alarm items may fall into two different judge examples. We cannot do the next version, and can only try to do it in the latter component such as alarm, but it is quite complicated.

Q: This max=3 means the same monitor item, for example cpu.busy can only trigger three alarms at most in a certain period of time?

A: Max means the maximum alarm times, for example you configured to alarm when cpu.idle is less than 5, set max as 3, then after it alarms for three times, it will not alarm even if it is less than 5, until cpu.idle is more than 5, it will show ok, then if it is less than 5 again, it will alarm again.

Q: Why was the alarm not triggered again when the same problem occurred in the monitored host after I labeled the occurred alarm as solved and did not recover the module of the alarm (newbie)?

A: The action of labeled as solved is not the same as what you think. This does not mean really solved, but it is to say, some alarms cannot recover by itself, so it stays in alarm-dash, the solved only means clearing up the remaining alarms in the alarm-dash.

Q: Can I write Expression like this, each (metric=net.port.listen port=8100 endpoint=1.2.3.4)?

A: Yes, you can.

Q: Do alarms have the system of automatic filtration?

A: There is no such system of automatic filtration. When strategy is configured in hostgrp, the strategy will add into the corresponding host. When we need to determine whether to alarm, data from a fake host cannot come up, therefore it will not trigger alarm. Furthermore, users do not wish to see that it will not alarm when no data come up, but we can fix it through alarms of nodata. 

Q: Alarms on year-on-year basis and with link relative ratio may have a problem which is false alarm case. Assume that A -> B -> C, among which B is the abnormal time point, it will alarm at point B, and remain normal at point C, and it will alarm again. Do you have any better ideas?

A: Alarms on year-on-year basis and with link relative ratio can make too many false alarms in practice. It is not practical, so we do not plan to support it, but only provide the function to alarm when traffic rises and falls sharply.

Q: How to cope when there is no alarm notification (emails, text messages, etc.) in Open-Falcon code?

A: Alarm writes the alarm email content in redis queue, sender will read and send, you can re-develop sender to make it send emails without http port, or you can refer to mail-provider and 'the community contributions' in the book.

##Data

Q: Once certain host is offline, we cannot search history data, can we?

A: No, we cannot. Once data do not update, we cannot search after 7 days. The period is cleared up in task index, and configuration port is not reserved, because the period seldom changes. If needed, we can change the delete period in the source code. @Fancy_Jin Shan (It should be noted that, it only means to clean index, the history data are still stored in the disk, when the index was updated again, index will create again automatically, history data can still be searched.)  

Q: Will the expansion or removal of machines in graph damage the original data?  

A: @Chen Kaijun Monitor data is fragmented and scattered on different graph examples according to Consistent Hashing Rule. The number of graph examples will severely impact data fragmentation result of Consistent Hashing. When there is expansion or reduction in graph group, the distribution rule for monitor data will be completely changed, old and new data cannot be both searched. Even if old data are not damaged, they cannot be searched. 

Q: Why is hostname unique but ip is not?
A: There is an id in host list. The id usually is for CMDB, and it is used for unique identification. If the hostname in cmdb is changed, it will be updated to monitor when it finds hostname is changed during updating. Agent can detect ip automatically, usually use intranet ip, but some machines may have several ips but no intranet ip, so it is not very easy to use ip automatically detected as the unique identification. 
Q: What is the meaning of the field come_from in grp list? 

A: It is used to distinguish the different sources of machine groups: the machine group may be added manually by users, or updated from other system internally, etc.

Q: Can I put history data in facon?

A: Data in falcon can only be added according to timestamp, and it cannot be inserted. With this feature, you can add data of yesterday and then add data of today sequentially at certain time point today. 

Q: How to reserve history data when expanding?

A: During expansion, history data cannot be moved. What we can do is to keep double writing data in expansion, for example persistently for a month, so that the function for users to view the picture will not be interrupted. Then history data will be moved to a host with a large disk, and build a dashboard particularly for history data searching. 

##Plugin

Q: When custom plugin is updated, do we need to restart agent to enter into force?

A: No, we do not. Curl -s "hostname:1988/plugin/update", then we can synchronize the plugin.

##Load Balancing

Q: Can Open-Falcon be the port agent? How to do load balancing?

A: Domain name resolution to ip. Visit by domain.name:port. There is no port agent. When transfer is more than one instance, it needs to configure transfer example list for the way of ip+port{6060, 8433}, which is inconvenient. It is suggested to use domain name to implement load balancing. Task cluster setting: https://github.com/open-falcon/task/blob/master/README.md

##Monitor

open-falcon is a monitor framework. Theoretically, for any software, as long as we can pack data into the format required by open-falcon, it can be monitored by open-falcon. For instance, the monitor of mysql we were discussing right now is actually to write a bunch of collection scripts and collect some indexes of mysql and push to open-falcon. The application itself can use similar method, write scripts to collect monitor performance indexes of the application and push to open-falcon. But if we all write our own scripts, there may be many duplications. The practice in Xiao Mi is: there is a commonly used jar package specific to java application, once maven leads in the jar package, some basic data can be collected, for example transfer latency, qps, etc. of certain port. It is a better practice to directly embed monitor data collection inside the service code. But we need to reduce invasion, and avoid block logic implement process of main services through the method of some queues or asynchronism. Data should be posted to local agent after collected. 

Q: Does Open-Falcon support Windows monitor? 

A: Yes. Please refer to 'Windows Host Monitor' in this book. 

Q: Please tell me, how does open-falcon monitor ports? I did not find the metirc of net.port.listen, and only found cpu,mem,net. 

A: You need to adapt a template, and then find the metric for port. The general structure is to configure template, the metric is net.port.listen, tag configuration as port=xxx, and the template correlates with hostgroup. 

Q: The error of index out of range always comes up in the agent of docker.

A: The problem is very typical. After investigation, we can trace the problem. Agent is running in the docker container, the docker container limits the number of cpu, agent gets the right number of cpu, for example the host is with 32 cores, the docker container limit is 2 cores, agent gets the number of cpu cores as 2. Them it prepares an array with length of 2 to hold the performance data for each cpu. However, agent is going to read /proc/stat, the content of /proc/stat inside the docker container is the same as the one inside the host, then we can see 32 cores, then index out of range.

Q: Is it suggested to use Agent for log analysis and monitoring?

A: Agent does not collect log files, and it is not a good practice to do log monitor this way. What I personally recommend is for service process to catch exception in code, and directly use alarm ports to alarm when there is any problem, when RD receives the alarm, it will research the log by itself. Xiao Mi may have a scribe group to collect logs. I do not know the specific procedures. 

##Safety Problems

Q: Is there any safety issue for all equipment to visit Transfer?

A: Transfer is deployed in intranet. Intranets with different IDCs are connected, so basically it has no issue of ACL when visiting Transfer.
Others

Q: How do components communicate with each other?

A: Transfer through jsonrpc tcp, codes are transferred according to binary system.

Q: What is used to draw the architecture map of Open-Falcon?

A: Mindmanager/PPT

Q: There are too many information and duplicated questions in the QQ group. Shall we build a BBS?

A: We seldom use BBS now. For common questions, you can folk github.com/open-falcon/book.git and then send PR, you can use Github issue as BBS for other questions. 

Q: What is the problem of too many open files?

A: You can check the limit for file descriptor through ulimit -a , if by default 1024 is too small, you can make it bigger through ulimit -n 65536.

Q: What is the plan of Open-Falcon for future?

A: https://github.com/XiaoMi/open-falcon/issues  I will update some things we are going to do soon and in future here (you can also make suggestions, if you have better ideas). For some of you who have the ability and some free time, you can claim one or more tasks, and we really welcome you to help write or provide related documents. Especially for those heavy users of open-falcon, we hope you can push your own improvements into the github warehouse of open-falcon :) (for instance, Meituan, Kingsoft Cloud, Cnkuai, Ganji, etc.) for those companies who connect to and use Open-Falcon, you can add related information to the comments of the issue in the following link https://github.com/XiaoMi/open-falcon/issues/4




