<!-- toc -->

# Nginx Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of Nginx can be done through [ngx_metric](https://github.com/GuyCheung/falcon-ngx_metric).

# Working Principle
Ngx_metric achieves the real-time analysis of nginx through the `log_by_lua` feature in lua-nginx-module, and save the intermediate result through `ngx.shared.DICT`. Finally extract the intermediate result, calculate, format and output the final result through external python script. The result in Falcon output format can be directly  pushed to Falcon-Agent.

# Help

For detailed instruction please wisit [ngx_metric](https://github.com/GuyCheung/falcon-ngx_metric).
