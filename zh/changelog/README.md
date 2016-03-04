## [0.1.0] 2016-03-08

### highlights
- `文档` API梳理和文档完善 http://docs.openfalcon.apiary.io `OpenFalcon-team @hitripod`
- `优化` graph集群扩容时，历史数据自动迁移 `OpenFalcon-team @yubo laiwei niean`
- `优化` 数据上报的最小间隔，可以在配置文件中更改 `OpenFalcon-team @niean`
- `新功能` 监控数据支持写入opentsdb `美团 OpenFalcon-team @Charlesdong`
- `新功能` 适配支持grafana `快网 OpenFalcon-team @hitripod`
- `新功能` 新增集群监控 `OpenFalcon-team @ulricqin`
- `新功能` 新增nodata监控 `OpenFalcon-team @niean`
- `新功能` agent内置URL监控 `@onlymellb`
- `优化` agent支持对多个transfer的负载均衡 `@cmgs`
- `优化` 往HostGroup中添加机器的时候如果发现机器名不存在，就直接插入host表 `@wkshare`
- `优化` dashboard绘图线条采用深颜色的配色方案 `美团 OpenFalcon-team @skyeydemon`

### changelog
- [agent] 优化：agent支持配置多个transfer地址 @CMGS https://github.com/open-falcon/agent/pull/37
- [agent] 优化：agent支持URL探测 @onlymellb https://github.com/open-falcon/agent/pull/27
- [alarm] bugfix：修改beego api不兼容引起的编译报错 @ulricqin
- [common] 新功能：增加tsdb的支持 @Charlesdong https://github.com/open-falcon/common/pull/2
- [dashboard] 优化：checkbox均支持使用shift快捷多选 @skyeydemon https://github.com/open-falcon/dashboard/pull/14
- [dashboard] 优化：绘图线条采用深颜色的配色方案 @skydemon https://github.com/open-falcon/dashboard/pull/13
- [dashboard] 优化：日期选择框高亮当前时间，方便用户选择 @iambocai https://github.com/open-falcon/dashboard/pull/12
- [dashboard] bugfix: 修复charts页面刷新时偶尔不出图的问题 @niean
- [fe] bugfix：修改beego api不兼容引起的编译报错 @hitripod
- [gateway] 优化：gateway支持配置多个transfer列表间负载均衡 @niean
- [gateway] 优化：gateway引入perfcounter，用来统计gateway自身的稳定性指标 @niean
- [graph] 数据上报的最小间隔，可以在配置文件中更改 @niean
- [graph] graph集群扩容时，历史数据自动迁移 @yubo laiwei niean https://github.com/open-falcon/graph/pull/14
- [hbs] 新功能：配置agent支持URL监控[url.check.health] onlymellb https://github.com/open-falcon/hbs/pull/4
- [nodata] 新模块：当某些采集项超时未上报数据时，如果配置了nodata策略则会生成一条默认的数据 @niean
- [portal] API: 增加了获取expression和strategy详情的API @modeyang
- [portal] 优化：往HostGroup中添加机器的时候如果发现机器名存在，就直接插入host表 @wkshare https://github.com/open-falcon/portal/pull/4
- [portal] 新功能：支持集群聚合监控 @ulricqin
- [portal] 新功能：支持nodata @niean
- [query] 增加API来支持Grafana展示 @hitripod https://github.com/open-falcon/query/pull/5
- [transfer] 数据支持写入opentsdb @Charlesdong https://github.com/open-falcon/transfer/pull/5
- [transfer] 数据上报周期的最小限制可配置 @niean https://github.com/open-falcon/transfer/pull/7
- [transfer] migrating的功能从transfer中去除 @laiwei https://github.com/open-falcon/transfer/pull/8
- [aggregator] 新模块：集群聚合监控 @ulricqin

----
## [0.0.5] 2015-08-20
- [agent] new feature: 增加了udp、du相关采集项
- [agent] bugfix: 修复了配置了新的插件需要重启agent的bug
- [agent] bugfix: 修复了reload配置文件，hostname改动可能无法生效的问题 @oiooj
- [alarm] bugfix: 修复了告警email中的换行问题
- [alarm] bugfix: 修复了告警分级配置项为空时处理不当的问题
- [alarm] enhancement: 修改http的默认端口为9912（之前为6060）
- [transfer] new feature: 新增了数据双写的功能，即transfer可以将同一份数据发送到后端的多个graph或者judge，用于容灾
- [transfer] enhancement: 发往judge的数据，按照时间戳做对齐和规整（与发往graph保持一致）
- [transfer] bugfix: 修复transfer返回给客户端的结果中latency单位错误的问题
- [graph] new feature: 新增了last API接口，可以返回指定counter最新的点
- [query] new feature: 新增了last API接口，可以返回指定counter最新的点
- [hbs] bugfix: 配合agent, 修复了配置了新的插件需要重启agent的bug
- [plugin] new feature: 新增了插件项目，里面有一些常用的插件脚本
- [gateway] 新增组件，解决网络分区后的监控数据回传问题。代码及功能描述，请移步到[这里](https://github.com/open-falcon/gateway)；Gitbook中没有该组件的描述。


## [0.0.4] 2015-06-09
- [alarm] bugfix:修复告警邮件中的换行问题
- [transfer] bugfix:当某个后端的graph宕机的时候，会引起transfer的发送能力下降
- [graph] bugfix: 当存储rrd数据文件的目录不存在或者没有读写权限的时候，程序作退出处理
- [fe] 新增了Golang版本的uic组件


## [0.0.3] 2015-06-02
 - [dashboard] bugfix: search counters by tags in screen
 - [judge] enhancement: clean stale data in memory


