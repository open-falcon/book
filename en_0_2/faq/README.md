<!-- toc -->


1. How does was the API file of Open-falcon v0.2 created? (The address of API file:  http://open-falcon.com/falcon-plus/)
> Users create yaml description file through Fi, then set up a static sites through Jekyll served by github. The source code of API site is saved at  https://github.com/open-falcon/falcon-plus/tree/master/docs. Users can open the gen_doc setting of API module. API will record the request and reply of the users and use them as references while writing API yaml file.

2. Does open-falcon v0.2 has administrator accounts?
> Users need to sign up in Dashboard. The first user signed up whose name is root will be considered as the super-administrator. The super-administrator can appoint others users as administrators.   

3. Can we forbid users to sign up in open-falcon v0.2 dashboard?
> Yes. Just change the value of `signup_disable` to `true` in the configuration file of API module and then reboot API.

4. Why will the error 'record not found' occasionally occur when Graph or Clone Screen is added in Dashboard of open-falcon v0.2?
> This is because of the bug of API, and we have fixed this bug at [pull-request #186](https://github.com/open-falcon/falcon-plus/pull/186). Please recompile the latest codes of API.

5. Can Open-falcon monitor the port of UDP?
> Yes. Please refer to [pull-request #66](https://github.com/open-falcon/falcon-plus/pull/66)。

6. How to separately compile and install API module in open-falcon v0.2?
> Please refer to [Environment Prepartion](https://book.open-falcon.com/zh_0_2/quick_install/prepare.html). All you need to do is to run `make.api` in the directory of falcon-plus.

7. Does open-falcon v0.2 haves administrator accounts?
> Yes. Please refer to [open-falcon grafana datasource](https://github.com/open-falcon/grafana-openfalcon-datasource).

8. Why does the alarm no longer be triggered after three alarms?
> There is a setting of maximum alarming frequency. For example, if you configure the policy that the alarm will be triggered when "cpu.idle" is less than 5 and the "max" is 3, then the alarm will be inactivated even when "cpu.idle" is less than 5 after 3 alarms. An OK message is sent when "cpu.idle" rises to no less than 5. After that, the alarm policy is effective again. 

9. I manually deleted some Endpoint list and some history data in endpoint_counter. And after that, the same index will not be inserted to MySQL.
> You can manually perform an index refreshing command of Graph, which means execute `curl -s http://127.0.0.1:6071/index/updateAll` on every Graph instance (provided the http port of Graph module is 6071). 

10. The disk of Graph (the directory of RRD file) is taking up more and more storage. How can I delete useless data?
> You can delete the RRD files that have not been updated in the last 7 days by executing these commands: `find . -name '*.rrd' -mtime +7 | xargs rm -f'`.

11. How to remove the out-dated monitored item in MySQL list?
> You can find and delete those items that have not been updated for a period according to the field t_modify in lists "graph.endpoint", "graph.endpoint_counter" and "graph.tag_endpoint". We recommend you to backup the entire backup list in case of any misoperations. Beside, we recommend you to trigger a full update of index.

12. When the alarm has reached the frequency limit, how to trigger the alarm again?
>  Three ways. One, increase the frequency limit. Two, change the threshold of alarm to recover it. Three, report a piece of normal data to recover it.

13. Dose Open-falcon has a English version?
> Dashboard of Open-falcon begins to support i18n since v0.2. For more information, please visit [here](https://github.com/open-falcon/dashboard/blob/master/i18n.md).

14. Can Open-falcon  send alarms to WeChat (or DingTalk)?
> The native open-falcon v0.2  supports sending alarms in WeChat. Please refer to  [alarm configuration](https://book.open-falcon.com/zh_0_2/distributed_install/mail-sms.html) and [WeChat Gateway Setup](https://github.com/Yanjunhui/chat) 。As for sending alarms in Dingtalk, please refer to [issue #134](https://github.com/open-falcon/falcon-plus/issues/134).

15. How can I immigrate the data when I increase the node of Graph during  Open-falcon expansion?
> For smooth expansion, please refer to [immigration of history data during Graph expansion](http://www.jianshu.com/p/16baba04c959)。

16. Can Open-falcon monitor Windows?
> Yes. Please refer to  [solutions for monitoring Windows](https://book.open-falcon.com/zh_0_2/usage/win.html)。

17. Can Open-falcon monitor the switches? 能监控交换机吗?
> Yes. Please refer to  [solutions for monitoring switches](https://book.open-falcon.com/zh_0_2/usage/switch.html) in the community.

18. Sender module is removed from Open-falcon since v0.2?
> Yes, we removed it to reduce the cost of maintenance and installation since v0.2 and integrated it with Alarm module.

19. Why is the data whose timestamp is smaller than the ones in Store is discarded by the same Counter in Graph? I wonder if it is because of the limitation of RRDtool so that users can only add data instead of inserting data. [issue](https://github.com/open-falcon/falcon-plus/issues/292)
> Yes. In the design mode of RRDtool, data can only be added, not be inserted. So, Falcon has sorted the data according to the timestamp in advance.

20. After a machine breakdown, why is there a latency and delay in Nodata alarm? [issue](https://github.com/open-falcon/falcon-plus/issues/294)
> The work logic of Nodata is: it will query data in Graph through API. If no data is found, then it will send a piece of preset data to this counter. The data from Nodata and user's operation are aligned by time. So we put a force latency code of one to two cycle in Nodata in case that the data from Nodata arrives at Graph earlier than normal data and makes normal data invalid.

21. Why are all the spaces in the tag field of Counter deleted? [issue](https://github.com/open-falcon/falcon-plus/issues/289)
> We consider it as a best regulation. In face, it is not for any purposes in code-level. We suggest you change the space to "-".

22. The disk inforamtion in alarm message is measured in bit. How can I change the unit? [issue](https://github.com/open-falcon/falcon-plus/issues/275)
> There is no definition of "unit " in Open-falcon. The meaning of data is strictly stick to the meaning when the data is reported by users. So please stick to the same concept.

23. When there are Chinese characters in the reported data, the error will occur when Graph is reading data: `errno: 0x023a, str:opening error` [issue](https://github.com/open-falcon/falcon-plus/issues/274)
> It has nothing to do with Chinese. This error occurs because Graph is reading a Counter that does not exist, which can be ignored.

24. How can data be reported and displayed in unit of days? [issue](https://github.com/open-falcon/falcon-plus/issues/271)
> User need to set step to 86400 and push the data every day. After that, the service endpoint will normalize the timestamp by the formula of "timestamp at service endpoint = int (current timestamp/86400)".

25. Where is the history alarm data of Open-falcon saved?
> In v0.1, the history data was discarded and not saved. We made an improvement in v0.2 so that history alarm data is saved in the database after being sent. Please refer to the list in the `alarms` database. Of course, these data is available at  [API](http://open-falcon.com/falcon-plus/#/alarm_eventcases_list).
