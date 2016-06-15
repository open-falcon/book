# 10.changelog

##[0.1.0] 2016-03-08

https://github.com/open-falcon/of-release/releases/tag/v0.1.0

http://www.jianshu.com/p/7751eb324a51

##highlights

* Document API combing and document improvement  http://docs.openfalcon.apiary.io OpenFalcon-team @hitripod
* Improvement when graph group expands, history data will move automatically OpenFalcon-team @yubo laiwei niean
* Improvement The minimum intervals when submitting data can be changed in configuration files OpenFalcon-team @niean
* New function Monitor data support write-in opentsdb Meituan OpenFalcon-team @Charlesdong
* New function Adapter support grafana Cnkuai OpenFalcon-team @hitripod
* New function Newly add group monitor OpenFalcon-team @ulricqin
* New function Newly add nodata monitor OpenFalcon-team @niean
* New function agent built-in URL monitor @onlymellb
* Improvement agent supports load balancing for multiple transfers @cmgs
* Improvement  When adding machines in HostGroup, if the machine name does not exist, we should directly insert host list @wkshare
* Improvement dashboard drawing lines use color schemes of deep color  Meituan OpenFalcon-team @skyeydemon

##changelog

* [agent] Improvement: agent supports to configure multiple transfer addresses @CMGS https://github.com/open-falcon/agent/pull/37
* [agent] Improvement: agent supports URL detecting @onlymellb https://github.com/open-falcon/agent/pull/27
* [alarm] bugfix: Change the error reports of compiling caused by incompatible of beego api @ulricqin
* [common] New function: add to the support of tsdb @Charlesdong https://github.com/open-falcon/common/pull/2
* [dashboard] Improvement: All checkboxes support to use shift shortcut multiple choice @skyeydemon https://github.com/open-falcon/dashboard/pull/14
* [dashboard] Improvement: use color schemes of deep color for drawing lines @skydemon https://github.com/open-falcon/dashboard/pull/13
* [dashboard] Improvement: Date check box highlights current time for the users to choose conveniently @iambocai https://github.com/open-falcon/dashboard/pull/12
* [dashboard] bugfix: Fix the problem of not showing the pictures for charts page sometimes when refreshing pages @niean
* [fe] bugfix: Change beego api the error reports of compiling caused by incompatible of beego api @hitripod
* [gateway] Improvement: gateway supports to configure load balancing between multiple transfer lists @niean
* [gateway] Improvement: gateway leads in perfcounter, it is used to prepare statistics of stability indexes for gateway itself @niean
* [graph] The minimum intervals for data submitting can be changed in configuration files @niean
* [graph] When graph groups expand, history data will be moved automatically @yubo laiwei niean https://github.com/open-falcon/graph/pull/14
* [hbs] New function: configure URL monitor supported by agent [url.check.health] onlymellb https://github.com/open-falcon/hbs/pull/4
* [nodata] New module: when data are not submitted overtime for some collection items, a piece of default data will be produced if nodata strategy is configured @niean
* [portal] API: Add API @modeyang to capture the details of expression and strategy
* [portal] Improvement: When adding machines in HostGroup, if the machine name does not exist, we should directly insert host list @wksharehttps://github.com/open-falcon/portal/pull/4
* [portal] New function: support group monitor @ulricqin
* [portal] New function: support nodata @niean
* [query] Add API to support demonstration of Grafana @hitripod https://github.com/open-falcon/query/pull/5
* [transfer] Data support write in opentsdb @Charlesdong https://github.com/open-falcon/transfer/pull/5
* [transfer] The minimum limit of data submission period can be configured @niean https://github.com/open-falcon/transfer/pull/7
* [transfer] The function of migrating can be deleted from transfer @laiwei https://github.com/open-falcon/transfer/pull/8
* [aggregator] New module: group monitor @ulricqin

##[0.0.5] 2015-08-20

* [agent] new feature: Add related collection items of udp, du
* [agent] bugfix: when we fix and configure new plug-in, we need to restart bug of agent
* [agent] bugfix: when fixing configuration files of reload, the problem that the change for hostname may not be valid @oiooj
* [alarm] bugfix: fix the problem of line feed in alarm emails
* [alarm] bugfix: fix the problem of mishandling when configuration items for alarm grading are empty
* [alarm] enhancement: change the default port of http as 9912 (used to be 6060)
* [transfer] new feature: add the function of data double-write, which means transfer can send the same data to more than one back-end graph or judge for disaster recovery
* [transfer] enhancement: data sent to judge should be aligned and put in order according to timestamp (stay the same with the ones sent to graph)
* [transfer] bugfix: fix the error of the unit for latency in the result transfer sent back to client side
* [graph] new feature: newly add ports for last API, can return to the newset point of assigned counter
* [query] new feature: newly add ports for last API, can return to the newest point of assigned counter
* [hbs] bugfix: cooperate agent, when we fix and configure new plug-in, we need to restart bug of agent
* [plugin] new feature: newly add plugin items and some common plugin scripts are inside
* [gateway] newly add components to solve the problem of monitor data transfer after network divisions. Code and function description can be found here; there is no description of the component in Gitbook.

##[0.0.4] 2015-06-09

* [alarm] bugfix: fix the problem of line feed in alarm emails
* [transfer] bugfix: when graph of certain backend is shutdown, it will cause the decrease of the sending ability for transfer 
* [graph] bugfix: when directory for storing rrd data files does not exist or provide read and write access, the program will exit
* [fe] newly add Golang version of uic component
[0.0.3] 2015-06-02
* [dashboard] bugfix: search counters by tags in screen
* [judge] enhancement: clean stale data in memory