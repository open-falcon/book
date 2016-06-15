# 4.9 Redis monitoring

In the part of Data Collection, we introduced common monitor data source. Open-falcon as a monitor frame can collect monitor index data of any system. The only thing is to change monitor data into the standard format of open-falcon.

The data collection of redis can be managed through collecting script redis-monitor or  redismon.

##Operating principle

redis-monitor is a cron. It runs a collecting script redis-monitor.py every minute, and the address of redis service is configured. redis-monitor is connected to redis instance to collect some monitor indexessuch as connected_clients, used_memory, etc., and then packed into a standard format data for open-falcon and posted to falcon-agent of this machine. falcon-agent provides a http port, and you can refer to the examples in Data Collection for its application method.

For instance, we have 1000 machines which are configured with Redis instance. We can deploy 1000 cron in the 1000 machines, i.e.: one-to-one correspondence with Redis isntance.