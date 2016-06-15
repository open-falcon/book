##HAProxy monitoring

We have introduced the usual monitoring data source in section of Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The data collection of HAProxy can be done by haproxymon.

##Operating principle

Haproxymon is a cron and the ```haproxymon.py``` is run every minute. Haproxymon collects the haproxy basic state information by the stats socket interface of haproxy, such as qcur、scur、rate etc., and then assemble to the normative format of open-falcon to post to the host falcon-agent. 

Falcon-agent provides a http interface, and as for the using method, please refer to the instances in Data Collection.
