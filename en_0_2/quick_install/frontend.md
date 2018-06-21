<!-- toc -->
----

## Environment Preparation

Please refer to [Environment Preparation](./prepare.md)

### Create a Working Directory
```
export HOME=/home/work
export WORKSPACE=$HOME/open-falcon
mkdir -p $WORKSPACE
cd $WORKSPACE
```

### Copy the Code of Frontend Modules
```
cd $WORKSPACE
git clone https://github.com/open-falcon/dashboard.git
```

### Install the Dependency Pack
```
yum install -y python-virtualenv
yum install -y python-devel
yum install -y openldap-devel
yum install -y mysql-devel
yum groupinstall "Development tools"


cd $WORKSPACE/dashboard/
virtualenv ./env

./env/bin/pip install -r pip_requirements.txt -i https://pypi.douban.com/simple
```

### Initialize the Database
Please refer to [Environment Preparation](./prepare.md)


### Change the Configuration
```
The configuration file of Dashboard is 'rrd/config.py` please change according to actual usage

## API_ADDR standd for the addess of backend API
module
API_ADDR = "http://127.0.0.1:8080/api/v1" 

## change PORTAL_DB_* according to actual usage; default user name is "root" and  default password is null
## change ALARM_DB_* according to actual usage; default user name is "root" and  default password is 
```

### Start as Developer
```
./env/bin/python wsgi.py

open http://127.0.0.1:8081 in your browser.
```

### Start in Production Environment
```
bash control start

open http://127.0.0.1:8081 in your browser.
```

### Stop Dashboard
```
bash control stop
```

### Check the Log
```
bash control tail
```

### Dashboard User Management 
```
Dashboard does not create any account including supervisor account by default. Users need to sign up in the webpage.
If you want to have an ultra supervisor account, please create an account whose name is root. (The first account whose name is root will be considered as ultra supervisor.) The ultra supervisor can allocate permissions to normal users.

Tips：Anyone who can open the page of Dashboard can create an account. So do not forget to disable the signning up feature of Dashboard after use. What you need to do is to change the value of item "signup_disable" to "true" in the API configuration file "cfg.json" then reboot API. When you want to create an account for someone, just recover the configuration and then disbale the item again.
```
