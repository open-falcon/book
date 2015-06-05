# 概述

Open-Falcon是一个比较大的分布式系统，有十几个组件。按照功能，这十几个组件可以划分为 基础组件、作图链路组件和报警链路组件，其安装部署的架构如下图所示，
![deploy-structure](http://www.tycloudstart.com/xiaomi/deploy/pict/falcon-deploy.png)
其中，基础组件以绿色标注圈住、作图链路组件以蓝色圈住、报警链路组件以红色圈住，橙色填充的组件为域名。OpenTSDB功能尚未完成。


大家安装的时候一定要有耐心，尽量按照本文档所述顺序安装，避免组件之间的依赖麻烦。
