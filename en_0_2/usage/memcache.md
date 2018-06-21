<!-- toc -->

# Memcache Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of Flume can be done through the script [memcached-monitor](https://github.com/iambocai/falcon-monit-scripts/tree/master/memcached).

## Working Principle

Memcached-monitor is a cron. It executes the script ```memcached-monitor.py``` once a minute that will automatically detect the port of Memcached , connect Memcached instance and collect some monitor metrics, like get_hit_ratio, usage and etc. Next, it convert the data to the format that is suitable for the Open-Falcon and post them to local falcon-agent. Falcon-agent provides an http port, and you can refer to the examples in [Data Collection](../philosophy/data-collect.md).

For example, if we deploy a Memcached instance in each of the 1000 machines, then we can deploy a cron instance in each machine that matches every Memcached instance.

What needs to be clarified is that the script ```memcached-monitor.py``` automatically finds Memcached port through ```ps -ef |grep memcached|grep -v grep |sed -n 's/.* *-p *\([0-9]\{1,5\}\).*/\1/p``` If user does not specify a port by configuring parameter ```-p``` when starting Memcached , the automatic detect of port will fail and user has to edit the script manually and specify a port.