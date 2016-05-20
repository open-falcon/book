## 支持 Grafana 视图展现

相较于 Open-Falcon 内建的 Dashboard，Grafana 可以很有弹性的自定义图表，并且可以针对 Dashboard 做权限控管、上标签以及查询，图表的展示选项也更多样化。本篇教学帮助您
做好 Open-Falcon 的面子工程！

### 开始之前

Open-Falcon 跟 Grafana 目前并不互相支持，所以您需要下面的PR

- Grafana [PR#3787](https://github.com/grafana/grafana/pull/3787) (支持到 v2.6 版) 
- Query [PR#5](https://github.com/open-falcon/query/pull/5)（已合并到最新的query代码中了，请检查您是否使用的是最新版)

### 设定 Datasource

当您取得包含上述 PR 的 Grafana 源代码之后，按照官方教学安装后依下述步骤编译：

1. 编译前端代码 `go run build.go build`
2. 编译后端代码 `grunt`
3. 执行 `grafana-server`

启动 Grafana 后，依照下图添加新的 Open-Falcon Datasource，需要注意的是我们这里使用的 URL 是在 falcon-query 中新增的 API。

![grafana1](https://raw.githubusercontent.com/hitripod/kordan.common.store/master/images/open-falcon/grafana_setting_1.png)

### 新增 Templating 变量

当 Open-Falcon 中已经有上百台机器时，一个个新增监控项到图表中是不切实际的，所以 Grafana 提供了一个 Templating 的变量让我们可以动态地选择想要关注的机器。

1. 上方设定点击 Templating
![grafana2](https://raw.githubusercontent.com/hitripod/kordan.common.store/master/images/open-falcon/grafana_setting_2.png)

2. 新增 Templating 变量
![grafana3](https://raw.githubusercontent.com/hitripod/kordan.common.store/master/images/open-falcon/grafana_setting_3.png)

### 新增圖表

有了 Templating 变量之后，我们就可以以它来代替 Endpoint 名称，选择我们关注的监控项，完成图表的新增。

![grafana4](https://raw.githubusercontent.com/hitripod/kordan.common.store/master/images/open-falcon/grafana_setting_4.png)
