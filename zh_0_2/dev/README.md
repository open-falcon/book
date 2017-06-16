# 环境准备

请参考[环境准备](quick_install/prepare.md)
# 自定义修改归档策略
修改open-falcon/graph/rrdtool/rrdtool.go

![](https://raw.githubusercontent.com/open-falcon/doc/master/img/custom-rra-1.png)
![](https://raw.githubusercontent.com/open-falcon/doc/master/img/custom-rra-2.png)

重新编译graph组件，并替换原有的二进制

清理掉原来的所有rrd文件（默认在/home/work/data/6070/下面)

# 插件机制
1. 找一个git存放公司的所有插件
2. 通过调用agent的/plugin/update接口拉取插件repo到本地
3. 在portal中配置哪些机器可以执行哪些插件
4. 插件命名方式：$step_xx.yy，需要有可执行权限，分门别类存放到repo的各个目录
5. 把采集到的数据打印到stdout
6. 如果觉得git方式不方便，可以改造agent，定期从某个http地址下载打包好的plugin.tar.gz

