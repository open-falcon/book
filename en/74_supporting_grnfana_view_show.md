# 7.4 Supporting Grnfana view show

##Supporting Grnfana view show

Compared to the Dashboard created by Open-Falcon, Grafana can self-define chart flexibly and control the permission, assign label as well as query aiming at Dashboard, and the show option is more diverse. The teaching help you do well in the show of Open-Falcon.

##Before start

Open-Falcon and Grafana don’t support each other at present, so you need the following PR

* Grafana PR#3787 ( v2.6 supported)
* Query PR#5（combined to the latest query code, please inspect if you are using the latest version)
* 
For more details please refer to Youku

##Set Datasource

When you get the abovementioned PR Grafana SC, install it as the official teaching, and compile it as follows:
1.Compile the front code go run build.go build

2.Comile the back code grunt

3.Execute grafana-server

After initiating Grafana, add new Open-Falcon Datasource as the following picture. Notice that the URL we use here is the newly added API in falcon-query. 

##picture

##Newly added Templating variable

It’s unrealistic to add new monitoring items to the chart one by one when there are already more than one hundred machines in Open-Falcon, so Grafana provides a variable of Templating so that we can dynamically choose the machine we want to pay attention to.

1.Set to click Templating 

2.Newly add Templating variable  

##Newly added chart

As for Templating, we can replace Endpoint name with it and choose the monitoring item we focus on to finish the adding of chart.

##picture