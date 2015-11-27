#DRRS

DRRS(Distributed Round Robin Server)即分布式环形数据库服务，该组件基于rrdtool，提供数据的环形存储和查询服务。在open falcon中功能和graph相同，用户可选择使用graph或者drrs。

drrs主要由两个模块组成：

1. master：路由服务器，根据路由规则路由外部应用请求到具体的RRD服务。

2. slave：rrd文件的存储服务器，其所在操作系统部署RRDTOOL，调用RRDTOOL接口进行create、update、fetch操作。

一个master对应一个或多个slave。

drrs的主要特点：

1. 轻量。drrs的基础配置只依赖于数据库。

2. 秒级扩容。扩容时用户只需新增slave服务器，并将其地址写入master的config文件，master周期性扫描config文件，读取最新的slave地址信息，进行外部请求的路由操作，以此来实现系统的秒级扩容。

3. 高效。drrs的master基本是全内存操作，slave自己实现的rrd cache功能，其性能是rrdcached的10倍+。

4. 高可用。slave服务器有多个，一个宕机对整个系统不会造成影响，同时，用户可部署多个master，并配以redis和zookeeper来实现信息共享，这样一个master宕机也不会对整个系统造成影响。同时，通过多个master的部署，还能大大提高系统的吞吐量。

##安装

drrs以rpm包的形式发布，master和slave是同一个可以执行程序，通过config文件的配置区分。

```bash
安装，umc_drrs会安装在/opt/wy/umc_drrs路径下。

rpm -ivh umc_drrs-1.1.1.x86_64.rpm
```
##配置说明

配置文件在/opt/wy/umc_drrs下的drrs.conf文件
```bash
serverType：0表示部署的是master,1表示部署的是slave，默认为1。

master_port:master向外提供服务的监听端口，部署master必填。

slave_port：slave向master提供服务的监听端口，部署slave必填。

splitPoint：每个slave最大可容纳的rrd文件数，一旦配置，不允许更改，部署master必填。

slaveGroup：slave地址，一个slaveGroup可包含一个或多个slave,master在一个slaveGroup内进行一致性hash路由，以解决数据热点问题。扩容时以slaveGroup为单位扩容。

dbAddr，dbPort，dbUser，dbPwd，dbName：数据库配置，部署master必填。

storageType：master中rrd文件信息保存的位置，0保存在内存，1保存在redis，部署多个master时必须存在redis，部署master必填。

redisAddr,redisPort：redis地址，storageType=1时必填。

zkAddr,zkPort,zkPath：zk地址，部署多个master必填。

rrdPath：rrd文件的存放位置，部署slave必填。

logDir：log存放位置。

logLevel：日志级别,0:所有日志；1:warning、error、fatal；2：error,fatal；3：fatal，默认为2。
```

##服务部署
###master
安装drrs并正确配置drrs.conf之后，即可部署master服务。
```bash
#导入数据库，脚本会默认创建名为umc的database,对应于drrs.conf中的dbName,如果不使用默认，请修改drrs_file.sql中的database。
mysql -hlocalhost -uroot -p < drrs_file.sql
#启动master，用安装目录下的install.sh启动
./install.sh 0
```
###slave
安装drrs并正确配置drrs.conf之后，即可部署slave服务。
```bash
#安装rrdtool
1.下载rrdtool源码，http://oss.oetiker.ch/rrdtool。
2.安装依赖包
yum install cairo-devel libxml2-devel pango-devel pango libpng-devel freetype freetype-devel libart_lgpl-devel
3.解压编译、安装
tar -xvf rrdtool-1.5.4.tar.gz
cd rrdtool-1.5.4
./configure --prefix=/usr/local/rrdtool && make && make install
rm -rf /usr/bin/rrd* && ln -s /usr/local/rrdtool/bin/* /usr/bin/
#启动slave，用安装目录下的install.sh启动
./install.sh 1
```
###补充说明
drrs服务部署好之后，请同步修改transfer和query中相应的配置项，使这两个组件可以寻址到drrs。









