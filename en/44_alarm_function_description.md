##Alarm function description

While configuring alarm strategies, open-falcon supports multiple alarm trigger function, for example, all(#3) diff(#10), etc., the numbers after # indicates that it is a newest historic point. For instance, #3 means the newest 3 points. 

```
all(#3): send alarm when all three newest points reach the threshold value
max(#3): send alarm when the maximum value of all three newest points reaches the threshold value
min(#3): send alarm when the minimum value of all three newest points reaches the threshold value
sum(#3): send alarm when the sum of all three newest points reaches the threshold value
avg(#3): send alarm when the average value of all three newest points reaches the threshold value
diff(#3): the newest point pushed (minuend) minus 3 newest points (3 subtrahends) equals 3 numbers, if one of them reaches the threshold value, then alarm will be sent
pdiff(#3): the newest point pushed minus 3 newest points equals 3 numbers, and divide the 3 numbers respectively by the 3 newest points (3 subtrahends) equals 3 values, if one of them reaches the threshold value, then alarm will be sent
```

The most commonly used function is ```all```, for instance cpu.idle ```all(#3) < 5```, means alarm will be sent when the value of cpu.idle is less than 5% for 3 consecutive times. 

It is not so easy to understand diff and pdiff. The design of diff and pdiff is to solve the problem of alarm for the sudden increase and decrease of flow. If you still cannot understand it, then you can only read the original codes: https://github.com/open-falcon/judge/blob/master/store/func.go

