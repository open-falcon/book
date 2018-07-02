<!-- toc -->

##[v0.2.0] 2017-06-17

> https://github.com/open-falcon/falcon-plus/releases/tag/v0.2.0

> http://www.jianshu.com/p/6fb2c2b4d030

> **Brand New Front-End**

- An integration of front-end modules in Open-Falcon: All the front-end components, including dashboard, screen, portal, alarm-dashboard, UIC, fe, links and etc.,are integrated in [dashboard](https://github.com/open-falcon/dashboard) component;
- Right control feature is added in the Dashboard;
- Specified Endpoint and Counter files along with corresponding RRD files can be deleted;
- Endpoint lists are displayed by default; you can turn pages in Endpoint lists and Counter lists;
- First class screen can be deleted in Dashboard;
- Parameters and contents of alarming Callback will be displayed in Dashbaord pages;
- Alarm channel via Wechat is supported;
- Alarm history is added in the Dashboard;

> **Integrated Back-End**

- Alarm history can be saved and displayed in Alarm;
- 「Alarm Combination」The features of module `links` are combined into integrated front-end Dashboard, reducing the cost of user's setting and maintenance；
- 「Alarm Sending」The features of module `sender` are combined into Alarm, reducing the cost of user's setting and maintenance;
- The features of Query are combined into Falcon-api component;
- The data of irregular reports can be stored;
- Agent can collect disk monitor data of specified disk mounting point via custom setting;
- Adding a default tag is supported in Agent, so that all data from this Agent will be sent along with the same tag by default;
- New alarm function is added in Judge: Alarm will goes off if certain conditions match limit times in past num cycles;

> **Long-waited Bug Fixing**

- Fixed a bug that Grafana didn't support capital letters;
- Fixed a bug that several Transfers with high availability written by Agent didn't work;
- Fix a bug that the time-out limit setting is unreasonable when Agent is sending data to Transfer; 

> **Brand New [RESTful API](http://open-falcon.com/falcon-plus)**：no more operations are hard to automate in Open-Falcon

- Brand new components Falcon-api is released; all the features in Falcon-plus are available in RESTful API; 
- The integration of most features in front-end Dashboard is achieved by [falcon-plus](https://github.com/open-falcon/falcon-plus) [api](http://open-falcon.com/falcon-plus);


## [0.1.0] 2016-03-08

> https://github.com/open-falcon/of-release/releases/tag/v0.1.0

> http://www.jianshu.com/p/7751eb324a51

### highlights
- `File` API filing and documentation http://docs.openfalcon.apiary.io `OpenFalcon-team @hitripod`
- `Optimization` Data migration will automatically start when Graph cluster is in expansion `OpenFalcon-team @yubo laiwei niean`
- `Optimization` Optimize the minimal time interval of data report, which can be changed in configuration   `OpenFalcon-team @niean`
- `New Feature` Opentsdb is supported in writing monitor data `美团 OpenFalcon-team @Charlesdong`
- `New Feature` Grafana is supported `快网 OpenFalcon-team @hitripod`
- `New Feature` Cluster monitoring `OpenFalcon-team @ulricqin`
- `New Feature` Nodata monitoring `OpenFalcon-team @niean`
- `New Feature` URL monitoring is built in Agent `@onlymellb`
- `Optimization` Multiple overload balacing is supported in Agent `@cmgs`
- `Optimization` If a machine name does not exist when machines are added in Hostgroup, the host list is inserted automatically
- `Optimization` Dark color scheme is used in curves in Dashboard diagrams`美团 OpenFalcon-team @skyeydemon`

### changelog
- [agent] Optimization：Multiple Agent address is aupported in setting @CMGS https://github.com/open-falcon/agent/pull/37
- [agent] Optimization：URL detection is supported in Agent @onlymellb https://github.com/open-falcon/agent/pull/27
- [alarm] bugfix：Fixed incompatibly in Beego api that causes compilation error @ulricqin
- [common] New Feature：Tsdb is supported @Charlesdong https://github.com/open-falcon/common/pull/2
- [dashboard] Optimization：Multiple object can be selected by using "Shift" key in Checkbox @skyeydemon https://github.com/open-falcon/dashboard/pull/14
- [dashboard] Optimization：Dark color scheme is used in curves in diagrams @skydemon https://github.com/open-falcon/dashboard/pull/13
- [dashboard] Optimization：Current time is highlighted in time setting for user's convenience @iambocai https://github.com/open-falcon/dashboard/pull/12
- [dashboard] bugfix: Fixed a bug that images are not correctly displayed every once in a while when Chart page is refreshed @niean
- [fe] bugfix：Fixed incompatibly in Beego api that causes compilation error @hitripod
- [gateway] Optimization：Overload balancing among multiple Transfer lists is supported in Gateway setting @niean
- [gateway] Optimization：Perfcounter is introduced in Gateway to evaluate the stability of Gateway @niean
- [graph] Minimal time interval of data sending can be changed in configuration files @niean
- [graph] Data migration will automatically start when Graph cluster is in expansion @yubo laiwei niean https://github.com/open-falcon/graph/pull/14
- [hbs] New Feature：URL monitoring is supported in Agent configuration [url.check.health] onlymellb https://github.com/open-falcon/hbs/pull/4
- [nodata] New Module：If nodata strategy is set, default data will be generated in data reporting when certain data collections time out @niean
- [portal] API: Acquiring Expression and Strategy information API @modeyang
- [portal] Optimization：If machine name does exist，the host list is inserted when machines are added in Hostgroup @wkshare https://github.com/open-falcon/portal/pull/4
- [portal] New Feature：Aggregated cluster monitoring is supported @ulricqin
- [portal] New Feature：Nodata is supported @niean
- [query] New API is added to support Grafana display @hitripod https://github.com/open-falcon/query/pull/5
- [transfer] Opentsdb is supported in data writing @Charlesdong https://github.com/open-falcon/transfer/pull/5
- [transfer] Minimal time interval of data report can be set @niean https://github.com/open-falcon/transfer/pull/7
- [transfer] Migrating feature is removed in Transfer @laiwei https://github.com/open-falcon/transfer/pull/8
- [aggregator] New Module：Aggregated cluster monitoring @ulricqin

----
## [0.0.5] 2015-08-20
- [agent] New Feature: Data collection concerned udp and du
- [agent] Bugfix: Fixed a bug that Agent needs to reboot when new plug-ins are set
- [agent] Bugfix: Fixed a bug that hostname in reload configuration file does not work after it is changed @oiooj
- [alarm] Bugfix: Fixed an issue of line breaks in alarm mail
- [alarm] Bugfix: Fixed a bug that Alarm cannot properly process data when alarm hierarchy configuration is null
- [alarm] Enhancement: Default port of http has been changed (from 6060) to 9912
- [transfer] new feature: Transfer can send the same copy of data to multiple Graph and Judge in the back-end for disaster tolerance
- [transfer] Enhancement: Data sent to Judge are aligned and adjusted according to timestamp (in accordance with data sent to Graph) 
- [transfer] Bugfix: Fixed a bug that the unit of latency in the result sent back to client by Transfer is not correct
- [graph] New feature: Last API port is added that can return to latest point in specified Counter
- [query] New feature: ast API port is added that can return to latest point in specified Counter
- [hbs] Bugfix: Fixed a bug that Agent needs to reboot when new plug-ins are set (in corelation with Agent)
- [plugin] New feature: New plugins are added along with some commonly used plugin scripts
- [gateway] New module is added that solves the issue of returning monitor data after network partition. For its codes and functional description, please visit [here](https://github.com/open-falcon/gateway) as the module is not described at Gitbook.


## [0.0.4] 2015-06-09
- [alarm] Bugfix: Fixed an issue of line breaks in alarm mail
- [transfer] Bugfix: Fixed a bug that the feeding ability of Transfer declines when one of the Graphs crashes
- [graph] Bugfix: Fixed a bug that the program exits when the file directory where RRD data are stored does not exist or does not have read-write access
- [fe] New Uic component in Golang version is added


## [0.0.3] 2015-06-02
 - [dashboard] bugfix: search counters by tags in screen
 - [judge] enhancement: clean stale data in memory


