Open-Falcon，整体可以分为两部分，即绘图组件、告警组件。这两个部分都可以独立工作，其中：

- [安装绘图组件](./graph_components.md) 负责数据的采集、收集、存储、归档、采样、查询、展示（Dashboard/Screen）等功能，可以单独工作，作为time-series data的一种存储展示方案。
- [安装告警组件](./judge_components.md) 负责告警策略配置（portal）、告警判定（judge）、告警处理（alarm/sender）、用户组管理（uic）等，可以单独工作。
- 如果你熟悉docker，想快速搭建并体验Open-Falcon的话，请参考 [使用Docker镜像安装Open-Falcon](https://github.com/frostynova/open-falcon-docker)
