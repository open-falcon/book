<!-- toc -->

# Q&A from QQ Group（373249123）

## About Alarm


Q: Can alarm mails list up top 3 programs that use the most network traffic when the traffic surpasses the threshold? Is it correct to write each expression like "endpoint=xxx metric=yyy tag1=tttt"? (By @聂安_小米)  

>A: They can. And you can configure the alarm policy by using each expression if the endpoint is not the name of the machine. If it is, then you can use the "hostgroup + template" configuration, which is more convenient for management. (You can still use the "hostgroup + template" configuration even when the endpoint is not the name of the machine, but you need to insert this endpoint into the hostgroup.)

Q: By the way, the maximum alarming frequency is 3. So where can I set the alarm time interval for the first 3 alarms? 

>A: First, let's suppose that the data are reported once a minute. Theoretically, the alarm will not be triggered anymore after it is triggered at the 3rd minute, the 6th minute and the 9th minute. But it seems too frequently for us. So we limit the minimum alarm time interval that is 5 minutes. For example: the 3rd minute, the 8th minute and the 13th minute. Link will not combine alarms, while Alarm will combine similar type of alarm in one minute. 

Q: How do I know that my policy is successfully synchronized?

>A: You can check the policy of the machine "host.test.01" and metric for "cpu.idle" at curl -s "judge-hostname:port/strategy/host.test.01/cpu.idle".

Q: Is it possible to let the alarm be triggered when the available disk space is less than 10% while it is less than 1TB?  Because it will happen in some big-capacity disks that 10% of the total space still equals many TBs.

>A: Sorry, no. Currently the architecture does not support policy combination.

Q: If I perform a ping test through two nodes, node A and B, then the two ping values will each come along with a tag, "ping_server=A" and "ping_server=b". Is it possible to configure an alarm policy that the alarm will be triggered only when node A and B are both not connected to the target machine? Beside, is it possible to configure an alarm policy that the alarm will be triggered when the connection fluctuation percentage of nginx with more than 1,000 connections surpass 20%? Because it may happen to some small nginx that the connection fluctuation percentage is 100% when the acctual connection number rises from 1 to 2. 
    
>A: This requirement can be settled by cluster monitor. The alarm is triggered only when two machines, not one, in a cluster breaks down. But currently, the architecture does not support policy combination of two different collection items. Because these two alarm items may be in two different Judge instances. And we will not add this feature in the next version. However, we will try to solve this problem in the next module of the link like Alarm, but it still can be quite tough.


Q: Does the number 3 in "max=3" means that the alarm will be triggered up for 3 times in a time period for the same monitor item, like "cpu.busy"? 

>A:Max means the maximum alarming frequency. For example, if you have configured the policy that the alarm will be triggered when "cpu.idle" is less than 5 and the "max" is 3, then the alarm will be inactivated even when "cpu.idle" is less than 5 after 3 alarms. An OK message is sent when "cpu.idle" rises to no less than 5. After that, the alarm policy is effective again.


Q: Why the alarm is not triggered when the same error occurs again in the monitored machine after I marked the alarm in "unsolved alarm" as "solved"? (from a newbie)

>A: "Marked as solved" is not like what people think. Actually it is not solved. Some alarms cannot be recovered so they are left in Alarm-Dash. What we call "solved" means removing the alarms left in Alarm-Dash. 

Q: Is the expression "each(metric=net.port.listen port=8100 endpoint=1.2.3.4)" correct?
    
>A: Yes, it is.
    
Q: Is there an automatic filtration system?
    
>A: No. When an hostgrp policy is configured, it will be added to the corresponding host. During alarm judging, it is impossible that false host sends data. Therefore, the alarm will not be triggered. Further more, users do not expect that the alarm is not triggered when no data are sent. Nodata alarm will can this problem.


Q: False alarm may happen based on comparison. In a link "A -> B -> C",  the alarm is triggered when time comes to point B. But it will be triggered again at point C even when the problem has recovered. Is there any better solutions?
    
>A: Indeed, there are many false alarms in real environment based on comparison. It is not that practical, so we are not going to support it. We only support alarm for drastic traffic fluctuation.
    
Q: There are no alarm notifications (mail, SMS and etc.) in the code of Open-Falcon. What can I do? 
    
>A: Alarm writes the content of alarm mail into the redis queue, and Sender will read and send it. What you can do is re-developing Sender, making it sending mails without calling http port, or referring to  [mail-provider](https://github.com/open-falcon/mail-provider) and "Community Support" in this book.

    
## About Data


Q: Is it true that the history data is not available once the machine is offline? (from @Fancy_金山)

>A: Yes, it is. Once the data are stopped from being updated for 7 days, they are not available for query. There is no configuration setting for the cycle of Task index deletion since it seldom needs to be changed. You can change the deletion cycle in the source code if necessary.

(Attention: What you delete is only the index. History data are still saved on the disk. An index will be created again when this item is updated again, and the history data is back for query.)


Q: Will the expansion of Graph or the removal of a machine damage the data? (from @陈凯军)

>A: The monitor data are sliced and printed in different Graph instances according to consistent hashing rules. The number of Graph instances will severely effect the result of data partition. When a Graph cluster expands or shrinks, the distribution rules will thoroughly change. New data and old data will not be inquired at the same time, even though the old data are not damaged.


Q: Why hostname not IP requires uniqueness?

>A: There is an ID in the host list, which is usually the ID of CMDB and the only indetification. If the hostname in CMDB changes, it is synchronized to monitoring during synchronization. IP is automatically detected by Agent, which is usually the IP in inner network. But some machine may have several IPs and no inner network IP. So it is not practical to set IP as the only identification.

Q: What does the field "come_from" in GRP list mean?

>A: It is used for distinguish the different sources of  ``Machine Group``. The machine group may be added by users or synchronized from the other inner system.

Q: Can I insert history data to Falcon?

>A: The data in Falcon can only be added according to timestamp and cannot be inserted. So you have to follow this rule: add yesterday's data at a time point today and then today's data.


Q: How can I save the history data during the disk expansion?

>A: The history data cannot be immigrated during expansion. For now, what you can do is keep data double-writing on, like for a month. So the users can keeping seeing the diagrams. Then transfer the history data to a machine with a big-capacity disk and set up a Dashboard for history data query.


## About Plugin

Q: Do I have to reboot Agent to activate the custom plugins after synchronization? 

>A: No. Just curl -s "hostname:1988/plugin/update" to synchronize the plugins.=


## About Loading Balacing

Q: Can Open-Falcon provide interface proxy? Can I get some advice on loading balacing?
    
>A: Resolve domain name to IP and visit through "domain.name:port". And we do not design interface proxy. When there are more than one instance in Transfer, the plan of ip+port{6060, 8433} needs the configuration of transfer instance list, which is not convenient. We recommend perform loading balancing through domain name.
    Task cluster configuration: https://github.com/open-falcon/task/blob/master/README.md


## About monitoring

>Open-falcon is a frame for monitoring. Theoretically, any software whose data are encapsulated in the format that Open-falcon requires, it can be monitored by Open-falcon. Take the MySQL monitor that everybody just discussed as an example. You just write a bunch of scripts for collecting some indexes of MySQL and push those data to to Open-falcon. It is the same with the application itself: write a bunch of scrpits for collecting some indexes of performance monitoring and push those data to to Open-falcon. But it is a waste of time and energy that everyone writes their scripts. The experiment in Xiaomi is preparing a general jar pack for java applications. Once it is introduced into maven, it will collecting some basic data, like the latency of a port, qps and etc. A better plan is embedding the monitor data collection in the business code but reducing the invasiveness by setting queues and asynchronization, so the logical execution process of main task is not blocked. Then just post the data to the Agent of the machine.

Q: Does Open-Falcon support Windows monitoring？

>A: Yes. Please consult the "Windows Monitoring" of this book.
    
Q: How does Open-Falcon monitor port? I cannot find the metric "net.port.listen but only cpu, mem and net?

>A: The metric of port will only show up after you configure a template. The overall architecture is: metric is "net.port.listen" and "port=xxx" in tag in configuration. Then associate the template with the hostgroup. The host in the hostgroup will spot out the port it needs to monitor through HBS, and report relevant data. 

Q: The error `index out of range` keeps reoccurring in the Agent of Docker.
    
>A: That is typical error. After investigation, here is the problem: Agent runs in the vessel of Docker that limits the number of CPU the Agent gets. Fro example, if the host machine has 32 cores and the core limit of Docker is 2, then Agent gets 2 cores. Agent prepares an array whose length is 2 for saving the performance data of each CPU. But, Agent next will read /proc/stat". What Agent sees in Docker vessel is usually the same as the one in the host machine, which is 32 cores in this case. So the error `index out of range` occurs.

Q: It it better to analyze and monitor log by using Agent?
    
>A: Agent do not collect log file, so it is not a sensible choice of log analysis. What I personally recommend is letting business program catch the exception by itself in the code and trigger the alarm by calling alarm port when an error occurs. RD will check the log after receiving  the alarm. Xiaomi company are supposed to have a cluster to gather the log, but I do not know the details.


## About Safety

Q: Is is safe that any machine can visit Transfer?

>A: Because Transfer is deployed in the inner network and different IDC inner networks are connected, so there is no ACL safety issues visiting Transfer.


## Others

Q: How does other modules communicate with each other?

>A: By binary code in jsonrpc and tcp transfer.
    
Q:The architecture diagram of the Open-Falcon is drawn by what tool?

>A: Mindmanager/PPT

Q: Can you build a forum because there are too many repetitive messages in the QQ group to spot out the useful information?

>A: The number of forum users is decreasing. You can fork github.com/open-falcon/book.git for common questions and post a PR. Then you can use Github issue as a forum.

Q: Why the error `too many open files` occurs?

>A: You can check the limitation of file descriptor through `ulimit -a`. If the value is too small, you can change it to a bigger value through `ulimit -n 65536`.

Q: Future plans of Open-Falcon?

>A: I will update the future plans at https://github.com/XiaoMi/open-falcon/issues . Welcome everybody to give your ideas! Anyone who has enough energy and time is  welcomed to claim one or multiple issues, help me write some articles, or contribute revelant documents. Especially, I hope the enterprise users of Open-Falcon, like Meituan, Kingsoft Cloud,  Fastweb and Ganji, can upload your improvement to the repository of Open-falcon at Github.
Companies that acesses and use Open-falcon and are using it can leave your information in the comment of  https://github.com/XiaoMi/open-falcon/issues/4
    
