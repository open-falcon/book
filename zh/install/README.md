# 概述

Open-Falcon是一个比较大的分布式系统，有十几个组件。按照功能，这十几个组件可以划分为 基础组件、作图链路组件和报警链路组件，其安装部署的架构如下图所示，

![deploy-structure](http://www.tycloudstart.com/xiaomi/deploy/pict/falcon-deploy.png)

其中，基础组件以绿色标注圈住、作图链路组件以蓝色圈住、报警链路组件以红色圈住，橙色填充的组件为域名。OpenTSDB功能尚未完成。

## 部分安装

Open-Falcon组件众多，如果你只想使用其绘图功能，只需要安装一下组件：
Graph/Transfer/Agent/Query/Dashboard/Task

如果还想使用报警功能，则需把剩下的组件也一并安装：
Sender/Web前端/Portal/Heartbeat Server/Judge/Alarm

如果还想使用报警合并功能，那Links也要安装

## Docker化的Open-Falcon安装

参看：https://github.com/frostynova/open-falcon-docker

## 视频教程教你安装

《[Open-Falcon部署与架构解析](http://www.jikexueyuan.com/course/1651.html)》

