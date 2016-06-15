##MySQL monitoring practice

In the part of Data Collection, we introduced common monitor data source. Open-falcon as a monitor frame can collect monitor index data of any system. The only thing is to change monitor data into the standard format of open-falcon.

The data collection of MySQL can be managed through mymon. 

##Operating principle

Mymon is a cron. It runs once every minute. In the configuration file, the database link address is configured. Mymon is connected to the database to collect some monitor indexes such as global status, global variables, slave status, etc., and then packed into a standard format data for open-falcon and posted to falcon-agent of this machine. falcon-agent provides a http port, and you can refer to the examples in Data Collection for its application method.

For instance, we have 1000 machines which are configured with MySQL instance. We can deploy 1000 cron in the 1000 machines, i.e.: one-to-one correspondence with instance in database. 

##Supplement

Remote monitor mysql instance If you want to collect mysql instance indexes in hostB through mymon of hostA, you can do it like this: "set the endpoint in the configuration file of mymon in hostA as the machine name of hostB, meanwhile, set mysql instance in hostB as the configuration item of [mysql]". When checking mysql indexes and adding strategies to mysql indexes, we need to find corresponding indexes for the machine name of hostB. 


