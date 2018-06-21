<!-- toc -->

# FAQ About Alarm

#### What should I do when the alarm never goes off even though I have configured policies?

1. Check the log of Sender, Alarm, Judge, Hbs, Agent and Transfer to see if there is any error
2. Visit the http webpage of Alarm through your browser to check if there isare any alarms needs to be reset,.If there is, then the alarmthey has been already generated. The reason why you didn't receive alarms is probably an error occurs at mail or SMS sending port. Then you need to check the API configuration in Sender.
3. Open Debug in Agent to see if it is still pushing data properly.
4. Check the configuration of Agent to see if the address of Heartbeat(HBS) and Transfer is correctly configured and enabled.
5. Check the configuration of Transfer to see if the address of Judge is correctly configured.
6. Judge provides an http port for debugging. We can use it to check if certain data has been correctly pushed. For example, if we want to check the data of "cpu.idle" of the machine "qd-open-falcon-judge01.hd", then run
```bash
curl 127.0.0.1:6081/history/qd-open-falcon-judge01.hd/cpu.idle
```
"127.0.0.1:6081" above means the http port of Judge.
7. Check if the time of server is synchronized through [ntp](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Understanding_chrony_and-its_configuration.html) or chrony.
8. Check if the HBS address of Judge is correctly configured.
9. Check if database address of HBS is correctly configured.
10. Check if alarm recipients are correctly configured in the policy template of Portal.
11. Check if the policy template in Portal configuration is bound to a HostGroup and the target machine happens to be in this HostGroup. 
12. Check UIC if you are added in the alarm recipient group
13. Check UIC to see if your contact information is correct.

#### An error occurs when I add a machine in HostGroup after creating a HostGroup in Protal page.

1. Check if the address of Heartbeat in correctly is configured and enable in Agent.
2. Check the log HBS.
3. Check if database address of HBS is correctly configured.
4. Check if the configuration of hosts is sync in HBS. HBS will only write host list when it is blank and you can only add a machine when there are data in host list.

