<!-- toc -->

# 大家最常问的问题

- Q: open-falcon v0.2的api文档是如何生成的？API文档地址 http://open-falcon.org/falcon-plus/
- A: 人工生成yaml描述文件，通过Jekyll来生成静态站点，使用github来serve，api站点的源码在 https://github.com/open-falcon/falcon-plus/tree/master/docs 这里。不过可以通过打开api组件的gen_doc选项，然后api会把请求和应答都记录下来，作为撰写api yaml文件的参考。

----
- Q: open-falcon v0.2 有管理员帐号吗？
- A: 可以通过dashboard自行注册新用户，第一个用户名为root的帐号会被认为是超级管理员，超级管理员可以设置其他用户为管理员。

----
- Q: open-falcon v0.2 dashboard 可以禁止用户自己注册吗？
- A: 可以的，在api组件的配置文件中，将`signup_disable`配置项修改为true，重启api即可。

----
- Q: open-falcon v0.2 dashboard 添加graph或者clone screen的时候，偶尔会出现'record not found'错误。
- A: 这是api的bug引起的，已经在 [pull-request #186](https://github.com/open-falcon/falcon-plus/pull/186) 修复，请重新编译最新的api代码。

----
- Q: open-falcon 可以监控udp的端口吗？
- A: 支持的，参考 [pull-request #66](https://github.com/open-falcon/falcon-plus/pull/66)。

----
- Q: open-falcon v0.2 如何单独编译安装api模块？
- A: 参考[环境准备](https://book.open-falcon.org/zh_0_2/quick_install/prepare.html)， 在falcon-plus的目录下，执行`make api`即可编译最新的api组件。

----
- Q: open-falcon 支持Grafana吗？
- A: 支持的，请参考[open-falcon grafana datasource](https://github.com/open-falcon/grafana-openfalcon-datasource)。

----
- Q: 为什么连续3次报警之后，不再发送报警了？
- A: 报警策略配置中，有一个最大报警次数的配置，比如你配置了cpu.idle小于5报警，max设置为3，那么报警达到3次之后即使仍然小于5也不会再报警了，直到接下来某次cpu.idle大于5了，就会报一个ok出来。以后如果又小于5了，那就会再次报警。

----
- Q: 手动删除了数据库中 endpoint表、endpoint_counter中的一些记录后，相同的指标不会再次插入到MySQL中了。
- A: 可以手工运行一次 graph 的索引刷新命令，即针对每个graph实例执行：`curl -s http://127.0.0.1:6071/index/updateAll` （这里假定graph模块http监听端口为6071）。

----
- Q: graph的磁盘占用越来越大，怎么清理掉无用的数据？（rrd文件目录越来越大）
- A:  在每台graph机器的rrd文件目录下面，执行如下命令 `find . -name '*.rrd' -mtime +7 | xargs rm -f'` 即可删除过去7天没有更新的rrd文件。

----
- Q: 如何清理MySQL表过期的监控指标项？
- A:  可以根据 graph.endpoint，graph.endpoint_counter，graph.tag_endpoint 三张表中的t_modify字段，找出很长一段时间都没有更新过的条目，进行删除。建议在操作数据库前，对数据库表做一个备份，免得误操作无法恢复，其次建议在操作前先触发一次索引的全量更新。

----
- Q:  报警达到最大次数之后，如何再次报警？
- A:  有三种方法：1是调大最大告警次数；2是修改告警阈值让该告警恢复；3是上报一个正常的值让该报警恢复。

----
- Q: open-falcon 有英文版本吗？
- A: open-falcon v0.2开始，dashboard部分支持i18n，点[这里](https://github.com/open-falcon/dashboard/blob/master/i18n.md)参与。

----
- Q: open-falcon 支持发送报警到微信吗？（钉钉呢？）
- A:  open-falcon v0.2 原生支持发送报警到微信，可以参考 [alarm配置](https://book.open-falcon.org/zh_0_2/distributed_install/mail-sms.html) 和 [微信网关搭建](https://github.com/Yanjunhui/chat) 。发送报警到钉钉，可以参考[issue #134](https://github.com/open-falcon/falcon-plus/issues/134)。

----
- Q：open-falcon 扩容增加graph节点，怎么让数据重新迁移？
- A：平滑扩容步骤，请参考 [graph扩容历史数据自动迁移](http://www.jianshu.com/p/16baba04c959)。

----
- Q: open-falcon 能监控windows吗？
- A:  可以的，请参考社区对 [windows监控的解决方案](https://book.open-falcon.org/zh_0_2/usage/win.html)。

----
- Q: open-falcon 能监控交换机吗？
- A: 可以的，请参考社区对 [交换机监控的解决方案](https://book.open-falcon.org/zh_0_2/usage/switch.html)。

----
- Q: open-falcon v0.2 中没有sender模块了吗？
- A: 是的，为了减少维护成本和安装成本，在open-falcon v0.2 中，移除了sender模块，将该功能集成在alarm模块中了。

----
- Q: 
- A: 

----
- Q: open-falcon 的报警历史是存储在什么地方的？ 
- A: 在v0.1版本中，历史报警信息是不存储的，发送后就丢弃了；在v0.2中，做了改进，报警发送后会将历史报警存储在数据库中，请参考 `alarms` 库下面的表。当然，也可以通过 [API](http://open-falcon.org/falcon-plus/#/alarm_eventcases_list) 来访问这些信息。
