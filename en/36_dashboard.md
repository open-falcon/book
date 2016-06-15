##Dashboard

Dashboard is the query interface for users. Here, users can see all the data pushed to Graph and view its trend chart.

##Dependency Installation

Dashboard is a Python project. When installing and deploying Dashboard, you need to install some dependent libraries. To install dependent libraries, perform the following steps:

```
# Install virtualenv. The root privileges are required. 
yum install -y python-virtualenv

# Install dependent libraries. No root privileges are required. You can use a common account. You need to go to the Dashboard directory for installation.
cd /path/to/dashboard/ 
virtualenv ./env
./env/bin/pip install -r pip_requirements.txt
```
For ubuntu users, the installation of mysql-python may fail. Please install dependent libmysqld-dev, libmysqlclient-dev, etc. by yourself.


##Service Deployment

Deploying Dashboard includes configuration modification, starting service, stopping service, etc. Before deploying Dashboard, you need to go to the deployment directory of Dashboard and perform the following steps:

```
# Modify configuration. Please refer to the following for meanings of configuration items.
vim ./gunicorn.conf
vim ./rrd/config.py

# Start service.
./control start

# Verify service.
# TODO

...
# Stop service.
./control stop
```
After the service started, you can view the running status of the service through logs. The address of the log file is ./var/app.log. You can visit the Dashboard homepage through http://localhost:8081 (it is assumed that the http listening port of Dashboard is 8081).

##Configuration Instruction
Dashboard has two configuration files that need to be modified: ./gunicorn.conf and ./rrd/config.py. Each field of ./gunicorn.conf is described as follows.

```
- workers,dashboard,the number of concurrent processes
- bind,dashboard,http listening port
- proc_name,process name
- pidfile,full name of the pid file
- limit_request_field_size,TODO
- limit_request_line,TODO
```

Each field of ./rrd/config.py is described as follows.

```
# Database configuration of dashboard
DASHBOARD_DB_HOST = "127.0.0.1"
DASHBOARD_DB_PORT = 3306
DASHBOARD_DB_USER = "root"
DASHBOARD_DB_PASSWD = ""
DASHBOARD_DB_NAME = "dashboard"

# Database configuration of graph
GRAPH_DB_HOST = "127.0.0.1"
GRAPH_DB_PORT = 3306
GRAPH_DB_USER = "root"
GRAPH_DB_PASSWD = ""
GRAPH_DB_NAME = "graph"

# Configuration of dashboard
DEBUG = True
SECRET_KEY = "secret-key"
SESSION_COOKIE_NAME = "open-falcon"
PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30
SITE_COOKIE = "open-falcon-ck"

# The address of query service
QUERY_ADDR = "http://127.0.0.1:9966"

BASE_DIR = "/home/work/open-falcon/dashboard/"
LOG_PATH = os.path.join(BASE_DIR,"log/")

try:
    from rrd.local_config import *
except:
    pass
```

##Supplementary Note

After Dashboard has started properly, you can configure the shortcut of Fe so that you do not need to enter ip:port separately to open Dashboard. After modifying the shortcut, you need to restart the Fe module.









