# 3.1Environment Preparation

##Environment Preparation

Environment preparation includes installing basic dependent components and preparing the installation environment for Open-Falcon.

##Dependent Components

###Install Redis

```
yum install -y redis
```
###Install MySQL
```
yum install -y mysql-server
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

##Installation Environment

All the back-end components of open-falcon are written using the Go language. In this section, we set up the Go language development environment and clone codes.

We use the 64-bit Linux as the development environment to keep consistent with the online environment. If you use a different environment, please solve the command differences between different platforms by yourself.

First, install the Go language development environment.

```
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

Then clone codes for future use.

```
cd $GOPATH/src
mkdir github.com
cd github.com
git clone --recursive https://github.com/open-falcon/of-release.git
```


