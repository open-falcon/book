# 环境准备

open-falcon的后端组件都是使用Go语言编写的，本节我们搭建Go语言开发环境，clone代码

我们使用64位Linux作为开发环境，与线上环境保持一致。如果你所用的环境不同，请自行解决不同平台的命令差异

首先安装Go语言开发环境：

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

接下来clone代码，以备后用

```bash
cd $GOPATH/src
mkdir github.com
cd github.com
git clone --recursive https://github.com/XiaoMi/open-falcon.git
```