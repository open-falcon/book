<!-- toc -->

# FAQ about Data Collection
Open-Falcon data collection includes [Graph Data] collection and [Alarm Data] collection. Here is how to check if the data collection works properly in these two links.


### How to check if [Graph Data] collection works properly?
The data link is `agent->transfer->graph->query->dashboard`. There is a http port in Graph that can check the link `agent->transfer->graph`. For example,  if the http port in Graph is 6071, then you can check by visiting:

```bash
# $endpoint and $counter are variables
curl http://127.0.0.1:6071/history/$endpoint/$counter

# If the data are sent without tags, then you should visit
curl http://127.0.0.1:6071/history/host01/agent.alive

# If the data are sent with tags, then you should visit
curl http://127.0.0.1:6071/history/host01/qps/module=graph,project=falcon
"module=graph" and "project=falcon" are tags
```
If those ports return void, that means Agent does not send data or an error occurs in Transfer.


### How to check if [Alarm Data] collection works properly?

The data link is `agent->transfer->judge`. There is an http port in Judge that can check the link `agent->transfer->judge`. For example,  if the http port in Judge is 6081, then you can check by visiting:

```bash
curl http://127.0.0.1:6081/history/$endpoint/$counter

# $endpoint and $counter are variables
curl http://127.0.0.1:6081/history/host01/cpu.idle

# counter=$metric/sorted($tags)
# If the data are sent with tags, then you should visit
curl http://127.0.0.1:6081/history/host01/qps/module=judge,project=falcon
```
If those ports return void, that means Agent did not send data or an error occurs in Transfer.

