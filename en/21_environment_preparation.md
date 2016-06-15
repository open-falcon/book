
##Environment Preparation

###Install Redis

```
yum install -y redis
```
###Install MySQL
```
yum install -y mysql-server

```

###create work directory

```
export HOME=/home/work
export WORKSPACE=$HOME/open-falcon
mkdir -p $WORKSPACE
cd $WORKSPACE
```

###Initialize the MySQL table structure

```
# All components of open-falcon can start without the root account. It is recommended that common accounts be used for installation to increase security. Here we use a common account work to install and deploy all components.
# However, the root account is required when yum is used to install some dependent lib databases.
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

##Download compiled component

We have compiled relevant component into binary version to make it easier to use. The binaries can only run on 64 bit Linux

Domestic users please click here to quickly download the compiled binary version.
```
DOWNLOAD="https://github.com/open-falcon/of-release/releases/download/v0.1.0/open-falcon-v0.1.0.tar.gz"cd $WORKSPACE

mkdir ./tmp#download
wget $DOWNLOAD -O open-falcon-latest.tar.gz
#uncompress
tar -zxf open-falcon-latest.tar.gz -C ./tmp/for x in `find ./tmp/ -name "*.tar.gz"`;do \
    app=`echo $x|cut -d '-' -f2`; \
    mkdir -p $app; \
    tar -zxf $x -C $app; \
    done
    ```
    
##Changelog

http://book.open-falcon.org/zh/changelog/README.html





