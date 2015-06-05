这个话题很难阐述，大家慢慢读：）

咱们从数据push说起，监控系统有个agent部署在所有机器上采集负载信息，比如cpu、内存、磁盘、io、网络等等，但是对于业务监控数据，比如某个接口调用的cps、latency，是没法由agent采集的，需要业务方自行push，其他监控数据，比如MySQL相关监控指标，HBase相关监控指标，Redis相关监控指标，agent也是无能为力，需要业务方自行采集并push给监控。

于是，监控Server端需要制定一个接口规范，大家调用这个接口即可完成数据push，拿agent汇报的cpu.idle数据举个例子：

```
{
    "metric": "cpu.idle",
    "endpoint": `hostname`,
    "tags": "",
    "value": 12.34,
    "timestamp": `date +%s`,
    "step": 60,
    "counterType": "GAUGE"
}
```

metric即监控指标的名称，比如disk.io.util、load.1min、df.bytes.free.percent；endpoint是这个监控指标所依附的实体，比如cpu.idle，是谁的cpu.idle？显然是所在机器的cpu.idle，所以对于agent采集的机器负载信息而言，endpoint就是机器名；tags待会再说；value就是这个监控指标的值喽；timestamp是时间戳，单位是秒；step是rrdtool中的概念，Falcon后端存储用的rrdtool，我们需要告诉rrdtool这个数据多长时间push上来一次，step就是干这个事的；counterType也是rrdtool中的概念，目前Falcon支持两种类型：GAUGE、COUNTER。

tags没有给大家解释，这是重点，为了便于理解，我再举个例子，某个方法调用的latency：

```
{
    "metric": "latency",
    "endpoint": "11.11.11.11:8080",
    "tags": "project=falcon,module=judge,method=QueryHistory",
    "value": 12.34,
    "timestamp": `date +%s`,
    "step": 60,
    "counterType": "GAUGE"
}
```

tags可以对push上来的这条数据打一些tag，就像写篇blog，可以打上几个tag便于归类。上例中我们push了一条latency数据，打了三个tag，传达的意思是：这个latency是调用QueryHistory这个方法的延迟，模块是judge，项目是falcon。

对于push上来的这种数据，我们配置监控就很方便了，比如：对于falcon-judge这个模块的所有方法调用的latency只要大于0.5就报警。我们可以配置一个expression（在portal中）：

```
each(metric=latency project=falcon module=judge)
```

阈值触发条件是：all(#1)>0.5

我们没有写method，所以对于所有method都生效，一条配置搞定！这就是tag的强大之处。

--重头戏分割线--

对于业务方自己push的数据，可以加各种tag来区分，传达更多信息。但是，对于agent采集的数据呢？cpu.idle、load.1min这种数据应该加什么tag呢？这种数据没有任何tag……那我们应该如何配置报警呢？总不至于一台机器一台机器配置吧……

```
each(metric=latency endpoint=host1)
```

公司才一万台机器，嗯，这酸爽……

tag，实际是一种分组方式，如果数据push的时候无法声明自己的分类，那我们就要**在上层手工对数据做分组**了。这，也就是HostGroup的设计出发点。

比如我们把falcon这个项目用到的所有机器放到一个HostGroup（名称姑且叫sa.dev.falcon，名称中包含部门、团队信息，这么命名不错吧）中，然后为sa.dev.falcon绑定一个策略模板，下面的所有机器就都生效了，扩容的时候增加的机器塞到sa.dev.falcon，也就自动生效了监控，机器下线直接从该组删掉即可。

总结，HostGroup实际是对endpoint的一种分组。

想象一下一条数据push上来了，我们应该怎么判断这条数据是否应该报警呢？那我们要做的就是找到这条数据关联的Expression和Strategy，Expression比较好说，无非就是看配置的expression中的tag是否是当前push上来的数据的子集；Strategy呢？push上来的数据有endpoint，我们可以看这个endpoint是否属于某几个HostGroup，HostGroup又是跟Template绑定的，Template下就是一堆Strategy，顺藤摸瓜：）

--忧伤的分割线--

但是，HostGroup是一个扁平结构，使用起来可能不是那么方便，举个例子。

- 我们可以把公司所有机器放到一个分组，配置硬件监控，比如磁盘坏，收到报警之后直接发送到系统组
- A同学和B同学共同管理了3个项目（包括falcon项目），可以把这三个项目的机器放到一个分组，配置机器负载监控，模板名称姑且叫sa.dev.common，配置一下cpu.idle、df.bytes.free.percent之类的，报警发给A、B两个同学
- falcon这个项目的磁盘io压力比较高，比其他两个项目io压力高，这是正常情况，比如平时我们配置disk.io.util大于40就报警，但是falcon的机器io压力大于80才需要报警，于是，我们需要创建一个新模板继承自sa.dev.common，然后绑定到falcon上，以此覆盖父模板的配置，提供不同的报警阈值
- falcon中有个judge组件，监听在6080端口，我们需要把judge的所有机器放到一个HostGroup，为其配置端口监控

OK，问题来了，如果judge扩容5台机器，这5台机器就要分别加到上面四个分组里！

略麻烦哈~所以我们内部使用falcon实际是配合机器管理系统的，机器管理对机器的管理组织方式是一棵树，机器加到叶子节点，上层节点也就同样拥有了这个机器。

所以，大家在用falcon的时候，规模如果比较小，就用扁平化的HostGroup即可。如果规模比较大，可能就需要做个二次开发，与内部机器管理系统结合了。如果你们内部没有机器管理，我们过段时间可能会提供一个树状机器管理系统，敬请期待。
