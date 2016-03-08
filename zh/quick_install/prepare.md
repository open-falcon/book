# 环境准备

### 安装redis
	yum install -y redis

### 安装mysql
	yum install -y mysql-server

### 创建工作目录
```bash
export HOME=/home/work
export WORKSPACE=$HOME/open-falcon
mkdir -p $WORKSPACE
cd $WORKSPACE
```

### 初始化mysql表结构
```bash
# open-falcon所有组件都无需root账号启动，推荐使用普通账号安装，提升安全性。此处我们使用普通账号：work来安装部署所有组件
# 当然了，使用yum安装依赖的一些lib库的时候还是要有root权限的。

git clone https://github.com/open-falcon/scripts.git
cd ./scripts/
mysql -h localhost -u root --password="" < db_schema/graph-db-schema.sql
mysql -h localhost -u root --password="" < db_schema/dashboard-db-schema.sql

mysql -h localhost -u root --password="" < db_schema/portal-db-schema.sql
mysql -h localhost -u root --password="" < db_schema/links-db-schema.sql
mysql -h localhost -u root --password="" < db_schema/uic-db-schema.sql
```


### 下载编译好的组件
** 我们把相关组件编译成了二进制，方便大家直接使用，这些二进制只能跑在64位Linux上 **

> 国内用户点这里高速下载编[译好的二进制版本](http://pan.baidu.com/s/1kTY121D)


```bash
DOWNLOAD="https://github.com/open-falcon/of-release/releases/download/v0.1.0/open-falcon-v0.1.0.tar.gz"
cd $WORKSPACE

mkdir ./tmp
#下载
wget $DOWNLOAD -O open-falcon-latest.tar.gz

#解压
tar -zxf open-falcon-latest.tar.gz -C ./tmp/
for x in `find ./tmp/ -name "*.tar.gz"`;do \
    app=`echo $x|cut -d '-' -f2`; \
    mkdir -p $app; \
    tar -zxf $x -C $app; \
done
```

### Changelog

	http://book.open-falcon.com/zh/changelog/README.html
