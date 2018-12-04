<!-- toc -->


# Practice of monitoring directory size and process detail

To collect the data of directory size and process details, we can use the scripts [falcon-scripts](https://github.com/ZoneTong/falcon-scripts)

## Collected Index

Below is the metrics:
| METRIC | NOTE |
|--------|------|
|du.bytes.used|directory size, byte|
|proc.cpu|process cpu, percent|
|proc.mem|process memory, byte|
|proc.io.in|process io input, byte|
|proc.io.out|process io output, byte|

## Working Principle

du.sh collects data by command du

proc.sh analyzes the data in /proc/$PID/status /proc/$PID/io and etc.
