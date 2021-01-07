<!-- toc -->

# 问与答from qq群（373249123）
## 报警


Q: 能不能当网络流量超过某个阀值的时候,发报警邮件把占用流量Top3的程序找出来?each (endpoint=xxx metric=yyy tag1=tttt)    expression可以这么写吗？ @聂安_小米

>A: 可以。如果endpoint不是机器名，可以用each表达式配置报警策略。如果endpoint是机器名，建议使用 hostgrp + template的方式 配置策略——这种方式 更加便于管理。（当然，endpoint不是机器名，也可以使用hostgrp + template的方式，只不过需要把该endpoint插入到hostgrp中）

Q: 捎带问下，最大报警次数为3，那前三次报警的时间间隔在哪里设置

>A: 比如你的数据是一分钟上来一次，理论上第三分钟，第六分钟，第九分钟分别报警三次就不再报警了。但是这样报警我们觉得太频繁，于是judge中有一个最小报警设置，默认是5分钟，即：两次报警之间至少间隔5分钟：第三分钟、第八分钟、第13分钟。Link不做告警合并，Alarm只合并一分钟内相同类型的报警。

Q: 怎么知道我的策略是否同步成功了？

>A: curl -s "judge-hostname:port/strategy/host.test.01/cpu.idle", 这个是查看 机器host.test.01 & metric为 cpu.idle 对应的策略。

Q: 我想判断磁盘空间少于10%并且剩余空间少于1TB 才报警，有些大容量的存储，10%的剩余空间可能还有好几 TB，这样可以实现吗？

>A: 目前架构不支持组合策略。

Q: 我现在做了一个 ping 检测，会通过2各节点 A,B 去 ping，对应的是2个 tag，ping_server=A,ping_server=b，我现在是想做一个报警项，当 A,B 节点  到目标主机都不通我才报警。还有例子，比如我想设置报警，nginx 的连接数波动超过20% 而且链接数大于1000的才报警，可能有些机器业务的nginx 量很少，从连接数1到连接数2的波动就已经是100%了。
    
>A: 这个可以划归到集群监控的情况，一个集群两机器都宕机才报警，只有一个宕机不报警。组合策略如果是针对两个不同的采集项，那以现在的架构肯定是实现不了的，因为两个报警项可能落到了两个不同的 judge 实例上了。下个版本不会做，只能想办法在后面的组件比如 alarm 来搞，还挺复杂的


Q: 这个max=3 是指同一个监控项 比如cpu.busy 在一定时间内最多发三次报警吗？

>A: max表示最大报警次数，比如你配置了cpu.idle小于5报警，max设置为3那么报警达到3次之后即使仍然小于5也不会再报警了，直到接下来某次cpu.idle大于5了，就会报一个ok出来。以后如果又小于5了，那就会再次报警


Q: 为什么未恢复报警那个模块，我把出现的报警标记为已解决后，被监控的主机再次出现同样的问题，没有再次触发报警（新手）

>A: “标记为已解决” 这个动作，和大家理解的不一样。 这个并不是真正的已解决，只是说，有些报警自己没有办法恢复了，就留在alarm-dash里面了，这个solved只是把alarm-dash里面遗留的 清理掉


Q: Expression 可以这样写吗？each(metric=net.port.listen port=8100 endpoint=1.2.3.4)
    
>A: 可以
    
Q: 报警有自动过滤的机制吗？
    
>A: 没有自动过滤的机制，hostgrp里配置了策略就会添加到相应的host上。报警判断时，假的host不可能上来数据，因此也就不会触发报警。进一步讲，这种没有数据上报就不报警可能不是用户希望看到。可以透过nodata的报警搞定之。


Q: 同比环比报警可能会有一个问题, 就是误报的 case . 假设 A -> B -> C , 其中 B 是异常时间点. 到 B 这个点报警. 然后到 C 这个时间点恢复正常的话, 又会报警一次. 不知道大家有没有什么比较好的思路
    
>A: 同比环比的告警，用在实际的环境中，误报太多，缺乏实际的操作意义，所以我们不打算支持。只提供了流量突升突降的告警功能。
    
Q: 报警通知（邮件、短信、等等）在 Open-Falcon 代码里面没有，该如何实现？
    
>A: alarm将报警邮件内容写入redis队列，sender负责读取并且发送，你可以二次开发sender，让它不通过调用http接口实现邮件发送，或参考 [mail-provider](https://github.com/open-falcon/mail-provider) 以及本书中的'社区贡献'。

    
## 数据


Q: 一旦某台机器下线，历史数据就不能查询了？

>A: 是的。一旦数据不更新了，7天后就不能查询了。task索引清除的周期，没有留出配置接口，因为这个周期很少变更。如果需要，可以修改源码中的删除周期。@Fancy_金山 (需要注意的是，这个清理只是清理了索引，历史数据本身，仍然是存储在磁盘上的，当该指标再次被更新的时候，索引会自动再次创建，历史数据仍能被查询到)


Q: graph扩容或者摘除机器 ，会对原先数据有损坏吗？

>A: @陈凯军 监控数据，是按照一致性哈希规则，被分片、打在不同的graph实例上的。graph实例数量，会严重影响一致性哈希的数据分片结果。graph集群缩扩容时，监控数据的分布规则将彻底发生变化，新老数据不可能被同时查询到。即使老的数据没有被损坏，也是查询不到了。


Q: 为什么是 hostname 唯一，而不是ip唯一？

>A: host表中有个id，这个id通常是CMDB的id，用这个id做唯一标识，如果cmdb中hostname改了，同步的时候发现hostname变了就同步到监控了。ip是agent自动探测的，通常是取内网ip，但是有的机器可能有多个ip，可能没有内网ip，自动探测的ip作为唯一标识不是很好用。

Q: grp表的come_from字段是什么意思

>A: 用来区分 ``机器分组`` 的不同来源: 机器分组可能是用户手动添加的、也可能是从内部其他系统同步过来的、等等。

Q: 我能把历史数据放到falcon中吗？

>A: falcon中的数据，只能按照时间戳进行追加，不能插入。利用这个特性，你可以在今天的某个时间点，按照顺序把昨天的数据追加进来后，再追加今天的数据。


Q: 扩容时如何保留历史数据呢？

>A: 扩容的时候，历史数据无法迁移。现在能做的是，在扩容的时候，保持数据双写，比如持续一个月，这样让用户的看图功能，不要中断。然后把历史数据，全部移动到一个大容量磁盘的机器上，搭建一个dashboard，用来专门查询历史数据。


## 插件

Q: 自定义的插件同步后，是否要重启一下agent，才会生效？  

>A: 不需要。curl -s "hostname:1988/plugin/update"，这样就能同步插件。


## 负载均衡

Q: Open-Falcon 能做接口代理嗎？负载均衡建议怎么做呢？
    
>A: 域名解析到ip。用 domain.name:port访问。没有做接口代理。transfer大于1个实例时，ip+port{6060, 8433} 的方式需要配置transfer实例列表、不方便。建议用域名实现负载均衡。
    task叢集設定：https://github.com/open-falcon/task/blob/master/README.md


## 监控

>open-falcon是一个监控框架，理论上任何软件只要可以把数据组装为open-falcon要求的格式，就可以用open-falcon监控。比如刚才大家讨论的mysql的监控，其实就是自己写了一堆采集脚本去采集mysql的一些指标，然后push给open-falcon就可以了。应用本身也可以用类似的做法，写脚本采集应用的监控性能指标，然后push给open-falcon。不过每个人都自己写脚本，可能重复造很多轮子。小米内部的实践是：针对java的应用有一个通用jar包，只要maven引入这个jar包，就可以采集一些基础数据，比如某个接口的调用latency、qps等等。比较好的实践方式是把监控数据的采集直接嵌入业务代码。但是要减少侵入性，通过一些队列、异步之类的方式，不要阻断主要业务逻辑执行流程。收集到数据之后post给本机agent即可

Q: Open-Falcon 支持 Windows 监控吗？

>A: 支持，请见本书中的'Windows 主机监控'。
    
Q: 请教各位大神，open-falcon怎么监控端口？我没有搜到net.port.listen这个metirc，只有cpu,mem,net

>A: 要先配个模板，才会有port的metric,大概架构是，配置模板，metric是net.port.listen,tag里配置port=xxx，然后模板关联hostgroup，hostgroup里的host就会通过hbs获取到需要监测的port，然后探测上报

Q: docker 中的 agent 一直出现 `index out of range` 的错误
    
>A: 刚才这个问题很典型，经过追查，问题是这样的：agent运行在docker容器里，docker容器限制了cpu个数，agent拿到了正确的cpu个数，比如宿主机是32个核，限制docker容器为2个核，agent就拿到了cpu核数为2 。于是准备了一个长度为2的数组来放置各个cpu的性能数据。但是，接下来agent要去读取/proc/stat，docker容器内部看到的/proc/stat文件内容和宿主机一般无二，于是看到了32个核，于是index out of range。

Q: 建议用 Agent 做日志分析监控吗？
    
>A: Agent不采集日志文件，这么做日志监控不是个好的实践方式，我个人推荐的做法是业务程序自己在代码中catch住exception，有问题直接调用报警接口报警得了，RD 收到报警之后自己去查log，小米应该是有个scribe集群来收集日志，具体我也不清楚。


## 安全性問題

Q: 任何机器都可以访问 Transfer 会不会有安全性问题？

>A: Transfer部署在内网。不同IDC内网是联通的，因此访问Transfer基本没有ACL的问题


## 其他

Q: 组件之间怎么沟通的？

>A: jsonrpc  tcp传输，编码为二进制传输的
    
Q: Open-Falcon 的架构图是用什么画的？

>A: Mindmanager/PPT

Q: QQ 群信息太多、很多重复问题、又容易发散，是否建个论坛？

>A: 论坛现在大家比较少用了。常见问题大家可以 fork github.com/open-falcon/book.git 然后发 PR，其余的可以使用 Github issue 作为论坛使用。

Q: `too many open files` 是什么问题？
>A: 可以先透过 `ulimit -a` 检查 file descriptor 的限制，如果默认 1024 太小，可以透过 `ulimit -n 65536` 加大。

Q: Open-Falcon 的未来计划？

>A: https://github.com/XiaoMi/open-falcon/issues 一些接下来，要做的事情，我都会更新在这里（大家也可以提，思路更广一些） 有能力、有时间的朋友，可以认领一个或多个issue，或者配合撰写、贡献相关文档，都是热烈欢迎的。 特别是一些重度使用open-falcon的大户哈，希望能把你们自己的一些改进，也都推进到open-falcon的github仓库里哈：）【比如美团、金山云、快网、赶集之类的~~~】 接入并使用了Open-Falcon的公司，可以把相关信息追加到下面这个issue的评论中 https://github.com/XiaoMi/open-falcon/issues/4
    
