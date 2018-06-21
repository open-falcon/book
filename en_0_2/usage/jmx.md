<!-- toc -->

# Jmxmon Introduction
Jmxmon is a jmx monitor module based on Open-falcon. Along with the agent of open-falcon, it can collect the service state of any java process whose JMX service port is open and push the collected data to the service of Open-falcon.

## Main Feature

Collecting jvm information of java process through jmx, including gc time consuming, gc frequency, gc throughput, utilization rate of old generation, size of new generation promotion, active threads and etc.。

It does not hack the code of programm and cost little resources in the system.


## Collected Metrics
| Counters | Type | Notes|
|-----|------|------|
| parnew.gc.avg.time  | GAUGE  | average time consuming of each YoungGC(parnew) in a minute|
| concurrentmarksweep.gc.avg.time  | GAUGE  | average time consuming of each CMSGC in a minute|
| parnew.gc.count  | GAUGE  | counter of the YoungGC(parnew) in a minute  |
| concurrentmarksweep.gc.count  | GAUGE  | ounter of the CMSGC in a minute  |
| gc.throughput  | GAUGE  | total traffic ratio of GC (application running time/process total running time）  |
| new.gen.promotion  | GAUGE  | size of the new generation memory promotion in a minute  |
| new.gen.avg.promotion  | GAUGE  | average size of all new generation memory promotion in a minute  |
| old.gen.mem.used  | GAUGE  | memory usage of old generation老年代的内存使用量  |
| old.gen.mem.ratio  | GAUGE  | memory usage percentage of old generation  |
| thread.active.count  | GAUGE  | number of currently active thread  |
| thread.peak.count  | GAUGE  | peak number of thread  |

## Recommended Metrics in Monitor Alarm

The alarm metric and the threshold can be congifured flexibility according to their different features.

| Metric | Condition | Note|
|-----|------|------|
| gc.throughput  | all(#3)<98  | gc thtoughput rate stays below 98% will affect the performance  |
| old.gen.mem.ratio  | all(#3)>90  | it needs to be optimized that the old generation memory usage 10 over 90%  |
| thread.active.count  | all(#3)>500  | too many threads will affect the performance  |


# Help
Please visit [jmxmon](https://github.com/toomanyopenfiles/jmxmon) for more detailed instruction.

