# 7.2 Modify the drawing curve precision

##Modify the drawing curve precision

By default, Open-Falcon only saves the original monitoring data of the latest 12 hours, and the precision of them after 12 hours will be reduced and those data will be sampling and storing.

You may modify the drawing curve precision by the following steps if the default precision cannot satisfy your demand.

###Step 1, modify the RRA of graph plugin, and recompile the graph plugin 

The RRA of graph plugin is defined in the file graph/rrdtool/rrdtool.go, by default:
```
// RRA.Point.Sizeconst (
    RRA1PointCnt   = 720 // 1m one point stored 12h
    RRA5PointCnt   = 576 // 5m one point stored 2d
    // ...
)
func create(filename string, item *cmodel.GraphItem) error {
    now := time.Now()
    start := now.Add(time.Duration(-24) * time.Hour)
    step := uint(item.Step)

    c := rrdlite.NewCreator(filename, start, step)
    c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

    // Setting all kinds of filing strategies
    // 1 minute one point stored 12h
    c.RRA("AVERAGE", 0.5, 1, RRA1PointCnt)

    // 5m one point stored 2d
    c.RRA("AVERAGE", 0.5, 5, RRA5PointCnt)
    c.RRA("MAX", 0.5, 5, RRA5PointCnt)
    c.RRA("MIN", 0.5, 5, RRA5PointCnt)

    // ...

    return c.Create(true)
}
```
For example, if you want to save the original data for 90 days, you may modify the code as follows:
```
// RRA.Point.Sizeconst (
    RRA1PointCnt   = 129600 // 1m one point stored 90d, value of 90*24*3600/60
)
func create(filename string, item *cmodel.GraphItem) error {
    now := time.Now()
    start := now.Add(time.Duration(-24) * time.Hour)
    step := uint(item.Step)

    c := rrdlite.NewCreator(filename, start, step)
    c.DS("metric", item.DsType, item.Heartbeat, item.Min, item.Max)

    // Setting all kinds of filing strategies
    // 1 minute one point stored 90d
    c.RRA("AVERAGE", 0.5, 1, RRA1PointCnt)

    return c.Create(true)
}
```

###Step 2, eliminate the history data of graph

Eliminate the history data of all the index reported, i.e. eliminate all the rrdfile. If not, the precision modification reported can’t be effective.

###Step 3, redeploy the graph service

Compile the modified graph code, terminate the original graph service, and publish the modified graph.
The new precision can be effective only by modifying the graph plugin, regardless of other plugins of Open-Falcon. You can check the new drawing curve precision through Dashboard and Screen. 
Notice:

When modifying the drawing curve precision, you need to:

* Modify the graph SC and update the RRA
* Eliminate the history data of graph. If not, the precision modification reported can’t be effective.
* Any other plugins of Open-Falcon doesn’t need to be modified except graph.
* After the RRA is modified, the problem that drawing curve points are too many and the browser was down may occur. Please reasonably plan the RRA storage points, or adjust the period choice when executing drawing curve query. 