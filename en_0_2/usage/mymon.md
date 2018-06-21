<!-- toc -->

# MySQL Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of MySQL can be done through [mymon](https://github.com/open-falcon/mymon).

## Working Principle

Mymon is a cron that runs every minute. Its configuration has a database address. Mymon will be connected to that database and collect the state data, like global status, global variables, slave status and etc. Next it encapsulates them in the format that is suitable for Open-Falcon and post them to the local falcon-agent. Falcon-Agent provides an http port. You can refer to [Data Collection](../philosophy/data-collect.md) for its use.

If we deploy a MySQL instance in each on our 1000 machines, then we can deploy a cron in each machine, which means the database and the instance are matched one-to-one.

## Complementary Information
***Instance of Remote MySQL Monitor ***
If you want to collect the MySQL metric data of Host B through the mymon of Host A, here is the resolution: set the Endpoint configuration in mymon of Host A to the machine name of Host B and set the MySQL configuration to the MySQL instance of Host B. User needs to find the metric that is corresponding to the machine name of Host B while viewing MySQL metric and adding strategy to MySQL metric.