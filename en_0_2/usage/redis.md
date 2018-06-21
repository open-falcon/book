<!-- toc -->

# Redis Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of Redis can be done through the script [redis-monitor](https://github.com/iambocai/falcon-monit-scripts/tree/master/redis) or [redismon](https://github.com/ZhuoRoger/redismon).

## Working Principle

Redis-monitor is a cron that runs the script ```redis-monitor.py``` every minute. Its configuration has the address of redis service. Redis-monitor will be connected to redis instance and collect the monitor data, like connected_clients, used_memory and etc. Next it encapsulates them in the format that is suitable for Open-Falcon and post them to the local falcon-agent. Falcon-Agent provides an http port. You can refer to [Data Collection](../philosophy/data-collect.md) for its use.

For example, if we deploy a Redis instance in each of the 1000 machines, then we can deploy a cron in each of the 1000 machine. Therefore, Redis instance and cron matches one-to-one.
