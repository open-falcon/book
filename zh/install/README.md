# 概述

Open-Falcon是一个比较大的分布式系统，有十几个组件。按照功能，这十几个组件可以划分为 基础组件、作图链路组件和报警链路组件，其安装部署的架构如下图所示，

![deploy-structure](http://www.tycloudstart.com/xiaomi/deploy/pict/falcon-deploy.png)

其中，基础组件以绿色标注圈住、作图链路组件以蓝色圈住、报警链路组件以红色圈住，橙色填充的组件为域名。OpenTSDB功能尚未完成。

## quick install

下面的各个小节每个小节安装一个组件，有些组件相互之间有依赖关系，均在文档中注明了。最快的无依赖安装顺序如下：

0. 根据环境准备所述，下载编译好的二进制
1. 邮件短信发送接口
2. Sender
3. Web前端
4. Portal
5. Heartbeat Server
6. Judge
7. Links
8. Alarm
9. Graph
10. Query
11. Dashboard
12. Transfer
13. Agent

