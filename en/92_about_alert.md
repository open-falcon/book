# 9.2 About alert

##Problems about alert
###After the strategy is deployed, there has been no alert. How to troubleshoot the problem?

1.Troubleshoot the log of sender、alarm、judge、hbs、agent、transfer

2.Access the http page of alarm by browser to see if there is unrecovered alert. If there is, the alert is generated and isn’t sent out. The interfaces of e-mail and message may go wrong and the deployed api in sender needs to be inspected.

3.Open the debug of agent to see if the push data is normal.

4.Inspect the agent configuration to see if the address of heartbeat(hbs) and transfer is correctly deployed and enabled.

5.Inspect the transfer configuration to see if the address of judge is correctly deployed.

6.Judge provides a http interface to debug and inspect if some data is correctly pushed up, for example, the cpu.idle data of qd-open-falcon-judge01.hd may be checked in this way: 
```curl 127.0.0.1:6081/history/qd-open-falcon-judge01.hd/cpu.idle```

7.Inspect if the time of server is synchronized, we may use ntp or chrony : 

**The said 127.0.0.1:6081 refers to the http port of judge.**

1.Inspect whether the hbs address judge deployed is correct

2.Inspect whether the database address hbs deployed is correct

3.Inspect whether the deployed strategy template in portal is deployed with the alert receiver

4.Inspect whether the deployed strategy template in portal is bound to some HostGroup and the aim machine is just in the HostGroup

5.Go to UIC and inspect whether it added itself in the alert receiver group

6.Go to UIC and inspect whether the contact information of itself is correct

###Creat a HostGroup in Portal page and report the error when adding machine to HostGroup

1.Inspect whether agent deployed the heartbeat address correctly and enabled

2.Inspect hbs log

3.Inspect the database address hbs deployed is correct

4.Inspect the deploy hosts of hbs is deployed to sync. Hbs will write the host table only when it is blank, and we may add machine on the page only when there is data in the host table
