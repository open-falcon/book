# 概述

Open-Falcon是一个比较大的分布式系统，有十几个组件。按照功能，这十几个组件可以划分为 基础组件、作图链路组件和报警链路组件，其安装部署的架构如下图所示，

![deploy-structure](https://raw.githubusercontent.com/open-falcon-niean/book/master/images/practice/deploy.png)

其中，基础组件以绿色标注圈住、作图链路组件以蓝色圈住、报警链路组件以红色圈住，橙色填充的组件为域名。OpenTSDB功能尚未完成。

## 二进制快速安装

请直接参考[quick_install](../quick_install/README.md)

## Docker化的Open-Falcon安装

参看：https://github.com/frostynova/open-falcon-docker

## 从源码安装

从源码，编译安装每个模块，就是本章的内容，请按照本章节的顺序，安装每个组件。

## 视频教程教你安装

《[Open-Falcon部署与架构解析](http://www.jikexueyuan.com/course/1651.html)》

