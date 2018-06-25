<!-- toc -->

#HAProxy Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of HAProxy can be done through [haproxymon](https://github.com/iask/haproxymon).

## Working Principle

Haproxymon is a cron that execute the collecting script ```haproxymon.py``` every minute. Haproxymon collects the basic state information of Haproxy, like qcur, scur and rate and etc., through the state socket of Haporxy, encapsulate them in the format that is suitable for Open-Falcon, and post them to the local falcon-agent. Falcon-Agent provides an http port. You can refer to [Data Collection](../philosophy/data-collect.md) for its use.