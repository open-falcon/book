##RabbitMQ monitoring

We have introduced the usual monitoring data source in section Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon

The data of RMQ can be collected by script rabbitmq-monitor.

##Operating principle

rabbitmq-monitor is a cron, and the script ```rabbitmq-monitor.py``` is run every minute, wherein RMQ username and password and so on are deployed. The script connects to the RMQ instance and collect some monitoring index such as messages_ready, messages_total, deliver_rate, publish_rate and so on, and then assemble to the normative format of open-falcon to post to the host falcon-agent. 

Falcon-agent provides a http interface, and as for the using method, please refer to the instances in Data Collection.

For example, we deployed 5 RMQ instance, and a cron can be run in every RMQ machine, i.e. it is one-to-one corresponded to the Memcached instance.
