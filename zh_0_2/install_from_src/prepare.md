# 环境准备
环境准备，包括安装基础的依赖组件 和 准备Open-Falcon的安装环境。

## 依赖组件
### 安装redis
	yum install -y redis
	
### 安装mysql
	yum install -y mysql-server
	
### 初始化mysql表结构
```bash
# open-falcon所有组件都无需root账号启动，推荐使用普通账号安装，提升安全性。此处我们使用普通账号：work来安装部署所有组件
# 当然了，使用yum安装依赖的一些lib库的时候还是要有root权限的。
export HOME=/home/work
export WORKSPACE=$HOME/open-falcon
mkdir -p $WORKSPACE
cd $WORKSPACE

git clone https://github.com/open-falcon/scripts.git     
cd ./scripts/
mysql -h localhost -u root -p < db_schema/graph-db-schema.sql
mysql -h localhost -u root -p < db_schema/dashboard-db-schema.sql

mysql -h localhost -u root -p < db_schema/portal-db-schema.sql
mysql -h localhost -u root -p < db_schema/links-db-schema.sql
mysql -h localhost -u root -p < db_schema/uic-db-schema.sql
```

## 安装环境
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
git clone --recursive https://github.com/open-falcon/of-release.git
```
