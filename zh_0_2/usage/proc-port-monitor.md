<!-- toc -->

# 引言

我们说falcon-agent是无需配置即可自动化采集200多项监控指标数据，比如cpu相关的、内存相关的、磁盘io相关的、网卡相关的等等，都可以自动发现，自动采集。

# 端口监控

falcon-agent编写初期是把本机监听的所有端口上报给server端，比如机器监听了80、443、22三个端口，就会自动上报三条数据：

```
net.port.listen/port=22
net.port.listen/port=80
net.port.listen/port=443
```

上报的端口数据，value是1，如果后来某些端口不再监听了，那就会停止上报数据。这样是否OK呢？存在两个问题：

- 机器监听的端口可能很多很多，但是真正想做监控的端口可能不多，这会造成资源浪费
- 目前Open-Falcon还不支持nodata监控，端口挂了不上报数据了，没有nodata机制，是发现不了的

改进之。

agent到底要采集哪些端口是通过用户配置的策略自动计算得出的。因为无论如何，监控配置策略是少不了的。比如用户配置了2个端口：

```
net.port.listen/port=8080 if all(#3) == 0 then alarm()
net.port.listen/port=8081 if all(#3) == 0 then alarm()
```

将策略绑定到某个HostGroup，那么这个HostGroup下的机器就要去采集8080和8081这俩端口的情况了。这个信息是通过agent和hbs的心跳机制下发的。

agent通过`ss -tln`拿到当前有哪些端口在监听，如果8080在监听，就设置value=1，汇报给transfer，如果发现8081没在监听，就设置value=0，汇报给transfer。

# 进程监控

进程监控和端口监控类似，也是通过用户配置的策略自动计算出来要采集哪个进程的信息然后上报。举个例子：

```
proc.num/name=ntpd if all(#2) == 0 then alarm()
proc.num/name=crond if all(#2) == 0 then alarm()
proc.num/cmdline=uic.properties if all(#2) == 0 then alarm()
```

proc.num表示进程数，比如进程名叫做crond的进程，其实可以有多个。支持两种tag配置，一个是进程name，一个是配置进程cmdline，但是不能同时出现。

那现在DEV写了一个程序，我怎么知道进程名呢？
首先要拿到进程ID，然后`cat /proc/$pid/status`，看到里面的name字段了么？falcon-agent就是根据这个name字段来采集的。此处有个坑，就是这个name字段最多15个字节，所以，如果你的进程名特别长可能被截断，截断之前的原始进程名我们不管，agent以这个status文件中的name为准。所以，你配置name这个tag的时候，一定要看一眼这个status文件，从这里获取name，而不是想当然的去写一个你自认为对的进程名。

再说说cmdline，name是从`/proc/$pid/status`文件采集的，cmdline是从`/proc/$pid/cmdline`采集的。这个文件存放的是你启动进程的时候用到的命令，比如你用`java -c uic.properties`启动了一个Java进程，进程名是java，其实所有的java进程，进程名都是java，那我们是没法通过name字段做区分的。怎么办呢？此时就要求助于这个`/proc/$pid/cmdline`文件的内容了。

cmdline中的内容是你的启动命令，这么说不准确，你会发现空格都没了。其实是把空格自动替换成`\0`了。不用关心，直接鼠标选中，拷贝之即可。不要自以为是的手工加空格配置到策略中哈，监控策略的tag是不允许有空格的。

上面的例子，`java -c uic.properties`在cmdline中的内容会变成：`java-cuic.properties`，无需把整个cmdline都拷贝并配置到策略中。虽然name这个tag是全匹配的，即用的`==`比较name，但是cmdline不是，我们只需要拷贝cmdline的一部分字符串，能够与其他进程区分开即可。比如上面的配置：

```
proc.num/cmdline=uic.properties if all(#2) == 0 then alarm()
```

就已经OK了。falcon-agent拿到cmdline文件的内容之后会使用`strings.Contains()`方法来做判断

听起来是不是挺复杂的？呵呵，如果你的进程有端口在监听，就配置一个端口监控就可以了，无需既配置端口监控、又配置进程监控，毕竟如果进程挂了，端口肯定就不会监听了。


