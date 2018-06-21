<!-- toc -->


# Practice of Monitoring Docker Container

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The data collection of docker container can be done through [micadvisor_open](https://github.com/open-falcon/micadvisor_open).

## Working Principle

Micadvisor-open is a module for monitoring the resource of docker container based on Open-Falcon. The data of CPU, memory, diskio and networkio will be pushed to Open-Falcon after being collected.

## Collected Index

| Counters | Notes|
|-----|------|
|cpu.busy|percentage of cpu usage|
|cpu.user|user-mode percentage of cpu usage|
|cpu.system|kernel-mode percentage of cpu usage|
|cpu.core.busy|usage of each cpu|
|mem.memused.percent|percentage of memory usage|
|mem.memused|amount of memory usage|
|mem.memtotal|total amount of memory|
|mem.memused.hot|hot usage of cpu|
|disk.io.read_bytes|bytes diskio reads|
|disk.io.write_bytes|bytes diskio write|
|net.if.in.bytes|incoming bytes of networkio|
|net.if.in.packets|incoming packets of networkio|
|net.if.in.errors|incoming errors of networkio|
|net.if.in.dropped|incoming droppings of networkio|
|net.if.out.bytes|outgoing bytes of networkio|
|net.if.out.packets|outgoing packets of networkio|
|net.if.out.errors|outgoing errors of networkio|
|net.if.out.dropped|outgoing droppings of networkio|

## Contributors
- mengzhuo: QQ:296142139; MAIL:mengzhuo@xiaomi.com 

## Complementary Information
- The lib database collected by another docker metric: https://github.com/projecteru/eru-metric

