<!-- toc -->

1. open-falcon v0.2的api文档是如何生成的？API文档地址 http://open-falcon.org/falcon-plus/
> 人工生成yaml描述文件，通过Jekyll来生成静态站点，使用github来serve，api站点的源码在 https://github.com/open-falcon/falcon-plus/tree/master/docs 这里。不过可以通过打开api组件的gen_doc选项，然后api会把请求和应答都记录下来，作为撰写api yaml文件的参考。

1. open-falcon v0.2 有管理员帐号吗？
> 可以通过dashboard自行注册新用户，第一个用户名为root的帐号会被认为是超级管理员，超级管理员可以设置其他用户为管理员。

1. open-falcon v0.2 dashboard 可以禁止用户自己注册吗？
> 可以的，在api组件的配置文件中，将`signup_disable`配置项修改为true，重启api即可。

1. open-falcon v0.2 dashboard 添加graph或者clone screen的时候，偶尔会出现'record not found'错误。
> 这是api的bug引起的，已经在 [pull-request #186](https://github.com/open-falcon/falcon-plus/pull/186) 修复，请重新编译最新的api代码。

1. open-falcon 可以监控udp的端口吗？
> 支持的，参考 [pull-request #66](https://github.com/open-falcon/falcon-plus/pull/66)。

1. open-falcon v0.2 如何单独编译安装api模块？
> 参考[环境准备](https://book.open-falcon.org/zh_0_2/quick_install/prepare.html)， 在falcon-plus的目录下，执行`make api`即可编译最新的api组件。

1. open-falcon 支持Grafana吗？
> 支持的，请参考[open-falcon grafana datasource](https://github.com/open-falcon/grafana-openfalcon-datasource)。

1. 为什么连续3次报警之后，不再发送报警了？
> 报警策略配置中，有一个最大报警次数的配置，比如你配置了cpu.idle小于5报警，max设置为3，那么报警达到3次之后即使仍然小于5也不会再报警了，直到接下来某次cpu.idle大于5了，就会报一个ok出来。以后如果又小于5了，那就会再次报警。

1. 手动删除了数据库中 endpoint表、endpoint_counter中的一些记录后，相同的指标不会再次插入到MySQL中了。
> 可以手工运行一次 graph 的索引刷新命令，即针对每个graph实例执行：`curl -s http://127.0.0.1:6071/index/updateAll` （这里假定graph模块http监听端口为6071）。

1. graph的磁盘占用越来越大，怎么清理掉无用的数据？（rrd文件目录越来越大）
> 在每台graph机器的rrd文件目录下面，执行如下命令 `find . -name '*.rrd' -mtime +7 | xargs rm -f'` 即可删除过去7天没有更新的rrd文件。

1. 如何清理MySQL表过期的监控指标项？
> 可以根据 graph.endpoint，graph.endpoint_counter，graph.tag_endpoint 三张表中的t_modify字段，找出很长一段时间都没有更新过的条目，进行删除。建议在操作数据库前，对数据库表做一个备份，免得误操作无法恢复，其次建议在操作前先触发一次索引的全量更新。

1.  报警达到最大次数之后，如何再次报警？
>  有三种方法：1是调大最大告警次数；2是修改告警阈值让该告警恢复；3是上报一个正常的值让该报警恢复。

1. open-falcon 有英文版本吗？
> open-falcon v0.2开始，dashboard部分支持i18n，点[这里](https://github.com/open-falcon/dashboard/blob/master/i18n.md)参与。

1. open-falcon 支持发送报警到微信吗？（钉钉呢？）
> open-falcon v0.2 原生支持发送报警到微信，可以参考 [alarm配置](https://book.open-falcon.org/zh_0_2/distributed_install/mail-sms.html) 和 [微信网关搭建](https://github.com/Yanjunhui/chat) 。发送报警到钉钉，可以参考[issue #134](https://github.com/open-falcon/falcon-plus/issues/134)。

1. open-falcon 扩容增加graph节点，怎么让数据重新迁移？
> 平滑扩容步骤，请参考 [graph扩容历史数据自动迁移](http://www.jianshu.com/p/16baba04c959)。

1. open-falcon 能监控windows吗？
> 可以的，请参考社区对 [windows监控的解决方案](https://book.open-falcon.org/zh_0_2/usage/win.html)。

1. open-falcon 能监控交换机吗？
> 可以的，请参考社区对 [交换机监控的解决方案](https://book.open-falcon.org/zh_0_2/usage/switch.html)。

1. open-falcon v0.2 中没有sender模块了吗？
> 是的，为了减少维护成本和安装成本，在open-falcon v0.2 中，移除了sender模块，将该功能集成在alarm模块中了。

1. graph中，对于同一个Counter，在收到数据的时候，如果该数据的Timestamp小于Store中第一个数据的Timestamp，则该数据会被丢弃，这是为什么呢？ 我想知道，这个地方出于这个考虑，是因为rrdtool的限制么，不能在Store中插入数据，只能追加数据吗？ [issue](https://github.com/open-falcon/falcon-plus/issues/292)
> 是的，在rrdtool的设计模式下，数据只能追加，不能插入。所以falcon在graph中提前对数据做了一个按照时间戳的排序。

1. 宕机后，nodata报警有一定时间的延迟和滞后，是什么原因？ [issue](https://github.com/open-falcon/falcon-plus/issues/294)
> nodata的工作逻辑是：通过api去graph中查询数据，如果没有查询到数据，则对这个counter补发设定后的值。nodata的补发数据 和 用户正常上报数据，都是靠时钟来对齐的，因此在nodata的代码中，强行延迟了一到两个周期，以免造成 nodata 补发的值先到达graph，这样就会造成正常的值失效。

1. counter中的tag字段，里面的所有空格都被去掉这是出于什么考虑呢? [issue](https://github.com/open-falcon/falcon-plus/issues/289)
> 这算作一个最佳实践的约定吧，并非代码实现层面的考虑。空格建议大家在上报之前自行替换为-号。

1. 报警信息里磁盘信息是以bit计数的，怎么控制单位呢？ [issue](https://github.com/open-falcon/falcon-plus/issues/275)
> open-falcon中没有「单位」的概念，数据都是遵从用户上报时的含义，理解上要保持一致。

1. 上报数据中有中文时，graph 读取数据报错：errno: 0x023a, str:opening error [issue](https://github.com/open-falcon/falcon-plus/issues/274)
> 这和中文没有关系，这个错误，应该是读取不存在的counter造成的，可以忽略。

1.  按天为单位，上报数据如何正确上报和展现？[issue](https://github.com/open-falcon/falcon-plus/issues/271)
> step要设置为86400，并且坚持每天push一次数据；push上去后，服务端会对时间戳做归一化，即服务端记录的时间戳 = int(当前时间戳/86400)。

1. open-falcon 的报警历史是存储在什么地方的？ 
> 在v0.1版本中，历史报警信息是不存储的，发送后就丢弃了；在v0.2中，做了改进，报警发送后会将历史报警存储在数据库中，请参考 `alarms` 库下面的表。当然，也可以通过 [API](http://open-falcon.org/falcon-plus/#/alarm_eventcases_list) 来访问这些信息。
