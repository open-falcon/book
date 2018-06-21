<!-- toc -->

# Explanation of Alarm Function

Open-Falcon supports many kinds of alarm triggerring function in alarm strategy configuration, like `all(#3)` and `diff(#10)`. Numbers after "#" stands for the latest history point. For example, `#3` means the latest three points.。

```bash
all(#3): trigger the alarm if the latest three points meet the threshold condition
max(#3): trigger the alarm if the maximum of latest three points meets the threshold condition
min(#3): trigger the alarm if the minimum of latest three points meets the threshold condition
sum(#3): trigger the alarm if the sum of latest three points meets the threshold
avg(#3): trigger the alarm if the average of latest three points meets the threshold
diff(#3): separately subtract the latest three points in the history from the newest pushed point, trigger the alarm if one of the three differences meets the threshold condition.
pdiff(#3): separately subtract the latest three points in the history from the newest pushed point, then divide the difference by the corresponding history point, trigger the alarm if one of the three quotients meets the threshold condition.
lookup(#2,3): trigger the alarm if two of the latest three points meet the threshold condition
```

The most common function is `all`, like cpu.idle `all(#3) < 5`, which means trigger the alarm if three consecutive values of cpu.idle are all less then 5%.

`lookup` is a non-consecutive alarm function used for tolerating the fluctuation of an index in a certain range. For example, if the cpu.busy of a machine has an obvious fluctuation, the `all(#1)>80` function would be too strict that triggers a lot of alarms confusing users. If user switch to `all(#3)>80` function, the alarm probably will never be triggered bacause the probability of three consecutive values being all very high is quite small. Therefore, users will never find the unstability of the system. In this circumstance, the better choice is `lookup(#3,5)`. Users will know that the value of cpu.busy fluctuates frequently beyond our tolerance.

It is not that easy to understand diff and pdiff function. Diff and pdiff are designed for the alarm of sudden data increase and decrease. If you still do not understand, please refer to the code at https://github.com/open-falcon/judge/blob/master/store/func.go