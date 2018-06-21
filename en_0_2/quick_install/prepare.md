<!-- toc -->

# Environment Preparation

### Install Redis
	yum install -y redis

### Install MySQL
	yum install -y mysql-server

**Attention: Make sure Redis and MySQL are enabled.**

### Initialize the Structure of MySQL List

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

**If you update v0.1.0 to the current v0.2.0，you only need to execute the following command：**

```
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
```

# Compile from the Source Code

First, make sure you have installed golang environment. If not, please refer to https://golang.org/doc/install

```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/

# make all modules
make all

# pack all modules
make pack

```

Then you will get a zipped pack of open-falcon-v0.2.0.tar.gz in the current directory, which means the compilation and packaging are successfully finished.

# Download the Compiled Binary Version

If you do not want to compile by yourself, you can download the compiled [Binary Version(x86 64-bit system)](https://github.com/open-falcon/falcon-plus/releases)。


The preparation is finished until this step. Unzip the binary pack open-falcon-v0.2.0.tar.gz in appropriate directory and save it for later use.
