# go开发环境搭建
```bash
cd ~
wget http://dinp.qiniudn.com/go1.4.1.linux-amd64.tar.gz
tar zxf go1.4.1.linux-amd64.tar.gz
mkdir -p workspace/src

echo "" >> .bashrc
echo 'export GOROOT=$HOME/go' >> .bashrc
echo 'export GOPATH=$HOME/workspace' >> .bashrc
echo 'export PATH=$GOROOT/bin:$GOPATH/bin:$PATH' >> .bashrc
echo "" >> .bashrc

source .bashrc
```

# clone代码

```bash
cd $GOPATH/src
mkdir github.com
cd github.com
git clone --recursive https://github.com/XiaoMi/open-falcon.git
```

# 编译一个组件(以agent为例)
```bash
cd $GOPATH/src/github.com/open-falcon/agent
go get ./...
./control build
```

# 自定义修改归档策略
1. 修改open-falcon/graph/rrdtool/rrdtool.go
![](https://raw.githubusercontent.com/open-falcon/doc/master/img/custom-rra-1.png)
![](https://raw.githubusercontent.com/open-falcon/doc/master/img/custom-rra-2.png)

2. 重新编译graph组件，并替换原有的二进制

3. 清理掉原来的所有rrd文件（默认在/home/work/data/6070/下面)

# 插件机制
1. 找一个git存放公司的所有插件
2. 通过调用agent的/plugin/update接口拉取插件repo到本地
3. 在portal中配置哪些机器可以执行哪些插件
4. 插件命名方式：$step_xx.yy，需要有可执行权限，分门别类存放到repo的各个目录
5. 把采集到的数据打印到stdout
6. 如果觉得git方式不方便，可以改造agent，定期从某个http地址下载打包好的plugin.tar.gz

