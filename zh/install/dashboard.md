# Dashboard
dashboard是面向用户的查询界面。在这里，用户可以看到push到graph中的所有数据，并查看其趋势图。

## 依赖安装

Dashboard是个Python的项目。安装&部署Dashboard时，需要安装一些依赖库。依赖库安装，步骤如下，

```bash
# 安装virtualenv。需要root权限。
yum install -y python-virtualenv

# 安装依赖。不需要root权限、使用普通账号执行就可以。需要到dashboard的目录下执行。
cd /path/to/dashboard/ 
virtualenv ./env
./env/bin/pip install -r `pip_requirements.txt`

```
对于ubuntu用户，安装mysql-python时可能会失败。请自行安装依赖libmysqld-dev、libmysqlclient-dev等。


## 服务部署

部署dashboard，包括配置修改、启动服务、停止服务等。在此之前，需要进入dashboard的部署目录，然后执行下列步骤

```
# 修改配置。各配置的含义，见下文。
vim ./gunicorn.conf
vim ./rrd/config.py

# 启动服务
./control start

# 校验服务
# TODO

...
# 停止服务
./control stop

```
服务启动后，可以通过日志查看服务的运行状态，日志文件地址为./var/app.log。可以通过```http://localhost:8081```访问dashboard主页(这里假设 dashboard的http监听端口为8081)。


## 配置说明
dashboard有两个需要更改的配置文件: ./gunicorn.conf 和 ./rrd/config.py。./gunicorn.conf各字段，含义如下

```bash
		- workers,dashboard并发进程数
		- bind,dashboard的http监听端口
		- proc_name,进程名称
		- pidfile,pid文件全名称
		- limit_request_field_size,TODO
		- limit_request_line,TODO
```

配置文件./rrd/config.py，各字段含义为

```python
# dashboard的数据库配置
DASHBOARD_DB_HOST = "127.0.0.1"
DASHBOARD_DB_PORT = 3306
DASHBOARD_DB_USER = "root"
DASHBOARD_DB_PASSWD = ""
DASHBOARD_DB_NAME = "dashboard"

# graph的数据库配置
GRAPH_DB_HOST = "127.0.0.1"
GRAPH_DB_PORT = 3306
GRAPH_DB_USER = "root"
GRAPH_DB_PASSWD = ""
GRAPH_DB_NAME = "graph"

# dashboard的配置
DEBUG = True
SECRET_KEY = "secret-key"
SESSION_COOKIE_NAME = "open-falcon"
PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30
SITE_COOKIE = "open-falcon-ck"

# query服务的地址
QUERY_ADDR = "http://127.0.0.1:9966"

BASE_DIR = "/home/work/open-falcon/dashboard/"
LOG_PATH = os.path.join(BASE_DIR,"log/")

try:
    from rrd.local_config import *
except:
    pass
```
## 补充说明
部署完成dashboard组件后，请修改dashboard组件的配置、使其能够正确寻址到query组件。