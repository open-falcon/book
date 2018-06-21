<!-- toc -->

# Flume Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of Flume can be done through [flume-monitor](https://github.com/mdh67899/openfalcon-monitor-scripts/tree/master/flume).

## Working Principle
```flume-monitor.py``` is a collecting script. Users only need to put it in the Plugin directory of the Falcon-Agent and bind corresponding pluging to the hostgroup in Portal. Falcon-agent automatically execute the script ```flume-monitor.py```. After the execution of the script, the result data of ```flume-monitor.py``` is output in json format which is read analysed by Falcon-Agent.

Java environment varible is supposed to be added in the configuration file when Flume is running. After booting up, Flume process will provide a monitor port. Users can collect the metrics provided by flume through http request. The script ```flume-monitor.py``` configures the Flume metric that needs to be collected, the module information that needs to be collected from flume port through http method, and output data in json format.

If we deploy three flumes on one machine, users can copy the script to three, edit the address of ```http url```, which makes it corresponding to the http port that flume monitors, and bind the plugin in Protal.
