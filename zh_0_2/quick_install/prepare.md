<!-- toc -->

# 环境准备

### 安装redis
	yum install -y redis

### 安装mysql
	yum install -y mysql-server

**注意，请确保redis和MySQL已启动。**

### 初始化MySQL表结构

```
cd /tmp/ && git clone https://github.com/open-falcon/falcon-plus.git 
cd /tmp/falcon-plus/scripts/mysql/db_schema/
mysql -h 127.0.0.1 -u root -p < 1_uic-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 2_portal-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 3_dashboard-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 4_graph-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
rm -rf /tmp/falcon-plus/
```

**如果你是从v0.1.0升级到当前版本v0.2.0，那么只需要执行如下命令：**

```
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
```

# 从源码编译

首先，请确保你已经安装好了golang环境，如果没有安装，请参考 https://golang.org/doc/install

```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/

# make all modules
make all

# pack all modules
make pack

```

这时候，你会在当前目录下面，得到open-falcon-v0.2.0.tar.gz的压缩包，就表示已经编译和打包成功了。

# 下载编译好的二进制版本

如果你不想自己编译的话，那么可以下载官方编译好的[二进制版本(x86 64位平台)](https://github.com/open-falcon/falcon-plus/releases)。


到这一步，准备工作就完成了。 open-falcon-v0.2.0.tar.gz 这个二进制包，请大家解压到合适的位置，暂时保存，后续步骤需要使用。
