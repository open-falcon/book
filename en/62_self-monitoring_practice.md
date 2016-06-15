# 6.2 Self-Monitoring Practice

##Self-Monitoring Practice

This section introduces some practical experience of Xiaomi in self-monitoring of the Open-Open-Falcon cluster.

##Overview

We call monitoring of the monitoring system self-monitoring. Self-monitoring requirements do not go beyond the business scope of monitoring. Like other systems, self-monitoring needs to do work in two fields: fault alarm and status demonstration. Fault alarm needs to identify faults in almost real time and notify the person in charge timely and requires high availability. Status demonstration is mainly used for prior prediction and post tracing and requires a lower magnitude of real time and availability than fault alarm. The following describes the two fields respectively.

##Fault Alarm

Fault alarm is relatively simple. We use the third-party monitoring component AntEye to monitor the health status of Open-Falcon instances.

Each component of Open-Falcon provides a self-monitoring interface that describes the availability of its own service, as described below. The AntEye service regularly inspects and aggressively calls the self-monitoring interface of each instance of Open-Falcon. If it finds that the interface of an instance does not return "ok" as agreed on, it regards this component as failed (as agreed on) and notifies the person in charge by short message or e-mail. To reduce the frequency of alarm notifications, AntEye adopts the simple alarm backoff policy and combines the contents of some alarm notifications based on actual conditions.
```
    # API for my availability
    Interface URL
        /health Inspects whether its own service runs properly

    Request method
        GET http://$host:$port/health
        $host Name or IP address of the computer where the service resides
        $port http.server listening port of the service

    Request parameter
        No parameter

    Returned result (string)
        "ok" (If "ok" is not returned, the service does not run properly)
```
The AntEye component aggressively pulls status data and loads monitoring instances, alarm recipient information, and alarm channel information by local configuration. This is to simplify alarm links and make the fault identification process as real-time and reliable as possible. The AntEye component is lightweight enough with few codes and simple functions, which can ensure the availability of a single AntEye instance; in addition, AntEye is stateless and can be deployed with multiple sets, which further ensures the high availability of the self-monitoring service.

In the same important network segment, 3+ AntEye instances usually need to be deployed, as shown in the figure below. We usually do not let AntEye perform monitoring across network segments because this will bring many false alarms at the network level. Multiple-set deployment will cause alarm notifications to be sent repeatedly, which is the cost of high availability; this repetition is acceptable based on our practical experience.

##picture

It is worth noting that the original fault identification function was a code snippet of the Task component of Open-Falcon. Later, to meet the requirements of multiple-set deployment, we moved the logic of fault identification out of Task and turned to use the independent third-party monitoring component AntEye.

##Status Demonstration

Status demonstration is to demonstrate the status data about the instances of each Open-Falcon component in graphical form to facilitate users' viewing. Because it does not require high real time and availability, we choose Open-Falcon to store and demonstrate the status data about itself (use Open-Falcon to monitor Open-Falcon), and the rest work is status data collection.
Most components of Open-Falcon provide an interface for querying service status data, as described below.
```
    # API for querying my statistics
    Interface URL
        /counter/all Returns all the status data

    Request method
        GET http://$host:$port/counter/all
        $host Name or IP address of the computer where the service resides
        $port http.server listening port of the service

    Request parameter
        No parameter

    Returned result
        // json format
        {
            "msg": "success", // "success" indicates that the request is processed successfully and all the other messages indicate failure.
            "data":[ // List of its own status data
                // Each item of status data includes the Name, Cnt, and Time fields and may include the Qps field.
                {
                    "Name": "RecvCnt",
                    "Cnt": 6458396967,
                    "Qps": 81848,
                    "Time": "2015-08-19 15:52:08"
                },
                ...
            ]    
        }
        ```
The Task component of Open-Falcon aggressively pulls the status data of each Open-Falcon instance through the above interface periodically; processes the status data and adapts them to the data formats required by Open-Falcon; and then pushes the adapted data to the local Agent; the local Agent forwards the data to the monitoring system Open-Falcon.

The Task component defines the features regarding status data collection using the collector item in the configuration file. as shown below.
```
    "collector":{
        "enable": true,
        "destUrl" : "http://127.0.0.1:1988/v1/push", // The adapted status data is sent to the local 1988 port (Agent receiver).
        "srcUrlFmt" : "http://%s/counter/all", // Format of the interface for status data query, where %s will be replaced with $hostname:$port in the cluster configuration item.
        "cluster" : [  
            // "$module,$hostname:$port", Indicates that the address $hostname:$port corresponds to a $module service.
            // In combination with the "srcUrlFmt" configuration, the interface for status data query "http://test.host01:6060/counter/all" can be obtained.
            "transfer,test.host01:6060", 
            "graph,test.host01:6071",
            "task,test.host01:8001"
        ]
    }
    ```
When adapting data, Task:Sets endpoint to the name of the host of data source $hostname ($hostname is the host name in a record of collector.cluster that Task collects and configures);Sets metric to $Name and $Name.Qps of original status data;Sets tags to module=$module,port=$port,type=statistics,pdl=falcon ($module and $port are the module name and port in a record of collector.cluster that Task collects and configures, and the other items are filled with fixed values);
Sets the data type to GAUGE;
Sets step to the data collection period of Task.
For example, Task that adopts the above collection configurations will perform the following adaptation:
```
    # One item of original status data, coming from "transfer,test.host01:6060"
    {
        "Name": "RecvCnt",
        "Cnt": 6458396967,
        "Qps": 81848,
        "Time": "2015-08-19 15:52:08"
   }
   

# After Task adapts data, two items of monitoring data are obtained.
    {
        "endpoint": "test.host01", // Host name in the configuration item "transfer,test.host01:6060" of collector.cluster configured by Task
        "metric": "RecvCnt", // $Name in the original status data
        "value": 6458396967, // $Cnt in the original status data
        "type": "GAUGE",    // Fixed as GAUGE
        "step": 60, // Data collection period of Task, 60s by default
        "tags": "module=transfer,port=6060,pdl=falcon,type=statistics", // The first two correspond to the module name and port in the configuration item "transfer,test.host01:6060" of collector.cluster of Task and the latter two are filled with fixed values.
        ...
    },
    {
        "endpoint": "test.host01",
        "metric": "RecvCnt.Qps", // $Name + ".Qps" in the original status data
        "value": 81848, // $Qps in the original status data
        "type": "GAUGE",
        "step": 60,
        "tags": "module=transfer,port=6060,pdl=falcon,type=statistics",
        ...
    }
    ```
    
We can only push one copy of status data to the monitoring system Open-Falcon (pushing multiple copies of data will cause overlapping and is not convenient for observation). Therefore, only one Task instance can be deployed in each network segment (similarly, it is not recommended to collect status data across network segments). Single point of deployment may have poor availability. However, the AntEye service will monitor the status of Task to identify faults of Task promptly, which can reduce the single point risk of the status data collection service to some extent.

After status data is written to Open-Falcon, we can customize the Screen page. The figure below shows the status data statistics page of Open-Falcon by Xiaomi. When customizing the page, you need to find the counter that you focus on. It can be searched using Dashboard, as shown in the figure below. Self-monitoring counters of different components are described in Appendix.
Screen status metrics regarding self-monitoring.


Customize your screen for self-monitoring status data.

##picture


With the screen for the status data of Open-Falcon itself, operation & maintenance become easy: Take 10 minutes to view these historical curves before starting work every morning and you can find problems that have occurred or even predict faults and evaluate capacity.

##Summary

For self-monitoring, related information is summarized as follows:

* A highly skilled doctor cannot treat himself. Fault monitoring of a large monitoring system usually needs the help of a third-party monitoring system (AntEye). The simpler and more independent the third-party monitoring system is, the better the monitoring effect is.
* Eat your own dog food. A system is easy to maintain only if its own status data is fully exposed. We try our best to make the storage container of status data common, make the interface for obtaining status data consistent, and make the status data collection service centralized, to facilitate inheritance and operation & maintenance. Currently, the program obtains its own status data through a process that is not elegant and with severe intrusion. We will highly appreciate it if you can offer your valuable suggestions.
 
##Appendix
###1. Open-Falcon Status Metrics
The following describes important status metrics (not all) as well as their meanings.
```
    ## transfer
    RecvCnt.Qps                        Qps for receiving data

    GraphSendCacheCnt                Cache length for sending data to Graph
    SendToGraphCnt.Qps                Qps for sending data to Graph
SendToGraphDropCnt.Qps            Qps for dropping data due to cache overflow when sending data to Graph
SendToGraphFailCnt.Qps            Qps for failing to send data when sending data to Graph

    JudgeSendCacheCnt                Cache length for sending data to Judge
    SendToJudgeCnt.Qps                Qps for sending data to Judge
    SendToJudgeDropCnt.Qps            Qps for dropping data due to cache overflow when sending data to Judge
    SendToJudgeFailCnt.Qps            Qps for failing to send data when sending data to Judge

    ## graph
    GraphRpcRecvCnt.Qps                Qps for receiving data
    GraphQueryCnt.Qps                Qps for processing Query requests
    GraphLastCnt.Qps                  Qps for processing Last requests
    IndexedItemCacheCnt                Number of cached indexes, specifically, number of monitoring metrics
    IndexUpdateAll                    Times of updating all indexes

    ## query
    HistoryRequestCnt.Qps            Qps for historical data query requests
HistoryResponseItemCnt.Qps        Qps for response items of historical data query requests
    LastRequestCnt.Qps                Qps for Last query requests

    ## task
CollectorCronCnt                 Times of collecting self-monitoring status data
    IndexDeleteCnt                    Times of cleaning spam indexes
    IndexUpdateCnt                    Times of updating all indexes

    ## gateway
    RecvCnt.Qps                        Qps for receiving data
    SendCnt.Qps                        Qps for sending data to Transfer
    SendDropCnt.Qps                    Qps for dropping data due to cache overflow when sending data to Transfer
    SendFailCnt.Qps                    Qps for failing to send data when sending data to Transfer
    SendQueuesCnt                     Cache length for sending data to Transfer

    ## anteye
    MonitorCronCnt                    Total times of judging status by self-monitoring
    MonitorAlarmMailCnt                Times of sending emails by self-monitoring alarm
    MonitorAlarmSmsCnt                Times of sending short messages by self-monitoring alarm
    MonitorAlarmCallbackCnt            Times of calling callbacks by self-monitoring alarm

    ## nodata
    FloodRate                            Percentage of occurrences of no data
    CollectorCronCnt                    Times of data collections
    JudgeCronCnt                        Times of no data judgments
    NdConfigCronCnt                     Times of pulling no data configuration
    SenderCnt.Qps                        Qps for sending simulation data
    ```