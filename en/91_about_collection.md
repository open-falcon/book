# 9.1 About collection


Open-Falcon data collection, including drawing data collection and alert data collection. We will introduce how to verify whether the data collection in two links is normal or not below.

##How to verify whether the drawing data collection is normal

The data link is: ```agent->transfer->graph->query->dashboard.``` There is a http interface of graph to verify the link ```agent->transfer->graph.``` For example, the http port of graph is 6071, and we may access the verification in this way:
```
# $endpointand$counter are variables
curl http://127.0.0.1:6071/history/$endpoint/$counter
# If the data reported are without tags, the access method is as follows:
curl http://127.0.0.1:6071/history/host01/agent.alive
# If the data reported are with tags, the access method is as follows, wherein the tags are module=graph,project=falcon
curl http://127.0.0.1:6071/history/host01/qps/module=graph,project=falcon
```
If null value is returned by the said interface, it means that agent doesn’t report data or there is an error in transfer service. 

##How to verify whether the alert data collection is normal

The data link is: ```agent->transfer->judge``` . There is a http interface of judge to verify the link ```agent->transfer->judge```. For example, the http port of judge is 6081, and we may access the verification in this way:
```
curl http://127.0.0.1:6081/history/$endpoint/$counter
# $endpointare $counterare variables, for example: 
curl http://127.0.0.1:6081/history/host01/cpu.idle
# counter=$metric/sorted($tags)# If the data reported are with tags, the access method is as follows, for example: 
curl http://127.0.0.1:6081/history/host01/qps/module=judge,project=falcon
```
If null value is returned by the said interface, it means that agent doesn’t report data or there is an error in transfer service.