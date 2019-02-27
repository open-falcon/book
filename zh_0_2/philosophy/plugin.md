<!-- toc -->

对于Plugin机制，叙述之前必须要强调一下：

> Plugin可以看做是对agent功能的扩充。对于业务系统的监控指标采集，最好不要做成plugin，而是把采集脚本放到业务程序发布包中，随着业务代码上线而上线，随着业务代码升级而升级，这样会比较容易管理。

要使用Plugin，步骤如下：

**1. 编写采集脚本**

用什么语言写没关系，只要目标机器上有运行环境就行，脚本本身要有可执行权限。采集到数据之后直接打印到stdout即可，agent会截获并push给server。数据格式是json，举个例子：

```bash
[root@host01:/path/to/plugins/plugin/sys/ntp]#./600_ntp.py
[{"endpoint": "host01", "tags": "", "timestamp": 1431349763, "metric": "sys.ntp.offset", "value": 0.73699999999999999, "counterType": "GAUGE", "step": 600}]
```

注意，这个json数据是个list哦

**2. 上传脚本到git**

插件脚本也是code，所以最好也用git、svn管理，这里我们使用git管理，公司内部如果没有搭建gitlab，可以使用gitcafe、coding.net之类的，将写好的脚本push到git仓库，比如上例中的600_ntp.py，姑且放到git仓库的sys/ntp目录下。注意，这个脚本在push到git仓库之前要加上可执行权限。

**3. 检查agent配置**

大家之前部署agent的时候应该注意到agent配置文件中有配置plugin吧，现在到了用的时候了，把git仓库地址配置上，enabled设置为true。注意，配置的git仓库地址需要是任何机器上都可以拉取的，即`git://`或者`https://`打头的。如果agent之前已经部署到公司所有机器上了，那现在手工改配置可能略麻烦，之前讲过的嘛，用[ops-updater](https://github.com/open-falcon/ops-updater)管理起来~

**4. 拉取plugin脚本**

agent开了一个http端口1988，我们可以挨个curl一下http://ip:1988/plugin/update 这个地址，这会让agent主动git pull这个插件仓库。为啥没做成定期拉取这个仓库呢？主要是怕给git服务器压力太大……大家悠着点用，别给人pull挂了……

**5. 让plugin run起来**

上一步我们拉取了plugin脚本到所有机器上，不过plugin并没有执行。哪些机器执行哪些plugin脚本，是在portal上面配置的。其实我很想做成，只要插件拉取下来了就立马执行，不过实际实践中，有些插件还是不能在所有机器上跑，所以就在portal上通过配置控制了。在portal上找到要执行插件的HostGroup，点击对应的plugins超链接，对于上例sys/ntp目录下的600_ntp.py，直接把sys/ntp绑定上去即可。sys/ntp下的所有插件就都执行了。

**6. 补充**

portal上配置完成之后并不会立马生效，有个同步的过程，最终是agent通过调用hbs的接口获取的，需要一两分钟。上例我们绑定了sys/ntp，这实际是个目录，这个目录下的所有插件都会被执行，那什么样的文件会被看做插件呢？文件名是数字下划线打头的，这个数字代表的是step，即多长时间跑一次，单位是秒，比如60_a.py，就是在通过命名告诉agent，这个插件每60秒跑一次。sys/ntp目录下的子目录、其他命名方式的文件都会被忽略。

**7. 插件如何传递参数** 

Open-Falcon 在 [PR #672](https://github.com/open-falcon/falcon-plus/pull/672) 中，对插件传递传递自定义参数进行了支持。在dashboard 中，配置 HostGroup 绑定插件时，可以支持针对单个脚本配置参数。

比如：`sys/ntp/30_xx.sh(a, "33 4", 'test.sh f\,d')`，表示对 hostgroup 绑定一个插件脚本`sys/ntp/30_xx.sh`, 并传递4个参数，多个参数之间用`,`分割，每个参数可以用双引号或者单引号括起来。如果参数中本身就包含逗号，可以使用 `\,` 来转义。

* 参数，只在绑定单个插件脚本时有效。如果绑定的是一个插件目录，传递的参数会忽略掉。
* 如果某个目录下的某个插件脚本，被单独绑定到某个hostgroup，同时该目录也被绑定到了这个hostgroup，这个插件脚本不会重复被执行，绑定目录时这个插件脚本会被忽略（也就是说，单个脚本的绑定会覆盖目录绑定方式下的同一个脚本）。
