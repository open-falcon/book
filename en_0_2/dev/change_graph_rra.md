<!-- toc -->

## Changing Precision of curves in Graph

Open-Falcon will only save the primary data in the last 12 hours. Data before that will be saved in samples with lower accuracy.

If the default accuracy does not meet your demand, you can change the accuracy of lines in graph following the steps below.

#### Step one: change the RRA of Graph module and recompile Graph module
the RRA of Graph module is defined in file at  graph/rrdtool/[rrdtool.go](https://github.com/open-falcon/graph/blob/master/rrdtool/rrdtool.go#L57), and here is the default setting:

```golang
// RRA.Point.Size
const (
	RRA1PointCnt   = 720 // 1m一个点存12h
	RRA5PointCnt   = 576 // 5m一个点存2d
	// ...
)

func create(filename string, item *cmodel.GraphItem) error {
	now := time.Now()
	start := now.Add(time.Duration(-24) * time.Hour)
	step := uint(item.Step)

	c := rrdlite.NewCreator(filename, start, step)
	c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

	// 设置各种归档策略
	// 1分钟一个点存 12小时
	c.RRA("AVERAGE", 0.5, 1, RRA1PointCnt)

	// 5m一个点存2d
	c.RRA("AVERAGE", 0.5, 5, RRA5PointCnt)
	c.RRA("MAX", 0.5, 5, RRA5PointCnt)
	c.RRA("MIN", 0.5, 5, RRA5PointCnt)
	
	// ...
	
	return c.Create(true)
}

```

For example, if you only want to save the primary date in the last 90 days, you can make following adjustment: 

```golang
// RRA.Point.Size
const (
	RRA1PointCnt   = 129600 // 1m一个点存90d，取值 90*24*3600/60
)

func create(filename string, item *cmodel.GraphItem) error {
	now := time.Now()
	start := now.Add(time.Duration(-24) * time.Hour)
	step := uint(item.Step)

	c := rrdlite.NewCreator(filename, start, step)
	c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

	// 设置各种归档策略
	// 1分钟一个点存 90d
	c.RRA("AVERAGE", 0.5, 1, RRA1PointCnt)

	return c.Create(true)
}
```

#### Step two: delete history data of Graph
Make sure you delete the history data of all indices that have been sent, which means delete all rrdfile, or the adjustment of accuracy won't take effect. 

#### Step three: redeploy Graph service
Compile the adjusted source code of Graph and shut down the old Graph service, then release the adjusted Graph.

New accuracy will take effect only after Graph module is  adjusted, not other modules in Open-Falcon. You can check the new curve accuracy in Dashboard and Screen.



### Attention:

When you change the curve accuracy in monitor graph

+ you need to change the source code of Graph and update RRA
+ you need to delete history data of Graph or the new adjustment of accuracy won't take effect
+ no modules in Open-Falcon need to be changed other than Graph module
+ After the RRA is modified, the problem that drawing curve points are too many and the browser was down may occur. Please reasonably plan the RRA storage points, or adjust the period choice when executing drawing curve query


