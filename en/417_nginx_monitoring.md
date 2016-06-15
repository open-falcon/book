##Nginx monitoring

We have introduced the usual monitoring data source in section of Data Collection. As a monitoring frame, open-falcon can collect monitoring index data in any system and it just need to organize the monitoring data to the normative format of open-falcon.

The data collection of Nginx can be done by ngx_metric.

##Operating principle

ngx_metric makes the real-time analysis of nginx request using the ```log by lua ```function of lua - nginx - the module , and stores the intermediate results with the help of ```NGX. shared. DICT```, and finally takes out the intermediate results to calculate, format and output through external python scripts. The falcon can be directly pushed to the falcon agent according to the output results falcon format.

##Help

For more detail please refer to: ngx metric