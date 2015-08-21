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


