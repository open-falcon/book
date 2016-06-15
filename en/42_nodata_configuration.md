##Nodata Configuration

To use Nodata, we need two configurations: Nodata configuration and strategy configuration. Now we will explain how to use the service provided by Nodata with an example. 

##User requirements

For all machines in the group ```cop.xiaomi_owt.inf_pdl.falcon_service.task```, when submission of their collecting index ```agent.alive``` is interrupted, users will be informed. 

##Nodata configuration 
Enter the homepage of Nodata, click the add button on the right top corner to add nodata configuration.  

##picture

After the above configuration, for all machines in the group named cop.```xiaomi_owt.inf_pdl.falcon_service.task```, when submission of their collecting index agent.alive is interrupted, nodata service will send a ```agent.alive``` monitor data with the value of ```-1.0``` to monitor system. 

##Strategy configuration

After the Nodata configuration, if there is any situation of data submission interruption, the default value in Nodata configuration will be sent. According to the default value, we can set up alarm. Once the default value is received, we believe there is an interruption in data submission (if the default value you set up equals the normal submission data, please change your Nodata configuration to make the default value different from normal value). Then you just need to bind the strategy to the group cop.```xiaomi_owt.inf_pdl.falcon_service.task```. 

##picture

##Notes

1.The name of configuration needs to remain the same in the system in order to manage Nodata configuration more easily.

2.The monitor instance of endpoint can be three types like machine group, machine name or others. But we can only choose one of them. The same type supports multiple records, but we suggest the numbers should be no more than 5. The records should be separated by line breaks with each record in a separate line. When choosing machine group, the system will support to extend to specific machine name with dynamic effect. When the monitor entity is not a machine name, we can only choose the type “others”

3.Monitor index metric.

4.Data tags. Multiple tags need to be separated by commas. Complete tags string must be filled, because nodata will match and filter monitor indexes strictly according to the tags string. 

5.Data type only support original value type GAUGE. Because nodata should only monitor “characteristic index” (like agent.alive). “Characteristic indexes” are all GAUGE type. 

6.Collecting step in seconds. Complete and real steps must be filled. If the field is incomplete or unreal, the nodata monitor will be reported wrongly or incompletely. 

7.Default must be different from submitted real data. For instance, the value range of cpu.idle is [0,100], then its nodata default can only be less than 0 or more than 100. Otherwise, there will be a wrong or incomplete report. 