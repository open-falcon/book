# Introduction

监控系统是整个运维环节，乃至整个产品生命周期中最重要的一环，事前及时预警发现故障，事后提供翔实的数据用于追查定位问题。监控系统作为一个成熟的运维产品，业界有很多开源的实现可供选择。当公司刚刚起步，业务规模较小，运维团队也刚刚建立的初期，选择一款开源的监控系统，是一个省时省力，效率最高的方案。之后，随着业务规模的持续快速增长，监控的对象也越来越多，越来越复杂，监控系统的使用对象也从最初少数的几个SRE，扩大为更多的DEVS，SRE。这时候，监控系统的容量和用户的“使用效率”成了最为突出的问题。

监控系统业界有很多杰出的开源监控系统。我们在早期，一直在用zabbix，不过随着业务的快速发展，以及互联网公司特有的一些需求，现有的开源的监控系统在性能、扩展性、和用户的使用效率方面，已经无法支撑了。

因此，我们在过去的一年里，从互联网公司的一些需求出发，从各位SRE、SA、DEVS的使用经验和反馈出发，结合业界的一些大的互联网公司做监控，用监控的一些思考出发，设计开发了小米的监控系统：Open-Falcon。


# Highlights and features

- **数据采集免配置**：agent自发现、支持Plugin、主动推送模式
- **容量水平扩展**：生产环境每秒50万次数据收集、告警、存储、绘图，可持续水平扩展
- **告警策略自发现**：Web界面、支持策略模板、模板继承和覆盖、多种告警方式、支持回调动作
- **告警设置人性化**：支持最大告警次数、告警级别设置、告警恢复通知、告警暂停、不同时段不同阈值、支持维护周期，支持告警合并
- **历史数据高效查询**：秒级返回上百个指标一年的历史数据
- **Dashboard人性化**：多维度的数据展示，用户自定义Dashboard等功能
- **架构设计高可用**：整个系统无核心单点，易运维，易部署

# Screenshots

**Dashboard Homepage**

![Dashboard Homepage](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/falcon-homepage.png)

**Dashboard Screen**

![Dashboard Screen](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/falcon-screen.png)

**大图**

![Dashboard Big chart](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/falcon-big-chart.png)

**Portal host group**

![Portal host group](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/portal-hostgrp.png)

**Portal template**

![Portal template](https://raw.githubusercontent.com/open-falcon/doc/master/screenshots/portal-tpl.png)

# Contributors

- 小米运维部: 博客 http://noops.me
- 来炜: https://github.com/laiwei 来炜没睡醒@微博 / hellolaiwei@微信
- 秦晓辉: http://ulricqin.com/ UlricQin@微博 UlricQin@Twitter
- 喻波: https://github.com/yubo x80386@微信
- 聂安: https://github.com/niean

# License

Copyright 2014-2015 Xiaomi, Inc. Licensed under the Apache License, Version 2.0: http://www.apache.org/licenses/LICENSE-2.0
