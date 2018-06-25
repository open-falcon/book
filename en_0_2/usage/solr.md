<!-- toc -->

# Solr Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of solr can be done through the script [solr_monitor](https://github.com/shanshouchen/falcon-scripts/tree/master/solr-monitor).

## Working Principle

Solr-monitor is a cron that runs the script ```solr_monitor.py``` every minute. It mainly collects the information of solr instance memory, cache hit and etc. Next it encapsulates them in the format that is suitable for Open-Falcon and post them to the local falcon-agent. 

The script can be deployed in each Solr instance with a cron that collects the data reguarly. Therefore, Solr instance and cron matches one-to-one.

If a server has multiple solr instances, user can change the ```servers``` property in ```solr_monitor.py``` to add the address of Solr instance to realize the local one-to-many data collection.
