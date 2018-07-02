<!-- toc -->

> **如何从OpenFalcon v0.1 平滑升级到 FalconPlus v0.2?**

###新增数据库表
```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/scripts/mysql/db_schema/
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
```

**NOTE:** 
1. falcon-plus 如何编译，请参考 [falcon-plus readme](https://github.com/open-falcon/falcon-plus) ;
2. 或者直接下载官方编译好的[二进制包](https://github.com/open-falcon/falcon-plus/releases)；

---
现在正式开始升级之旅:)

###1. 升级agent

用falcon-plus agent 二进制文件替换掉v0.1版本 agent 的二进制，同时注意修改agent的配置文件：

```
diff --git v0.1/agent/cfg.example.json v0.2/agent/cfg.example.json
--- v0.1
+++ v0.2

         "backdoor": false
     },
     "collector": {
-        "ifacePrefix": ["eth", "em"]
+        "ifacePrefix": ["eth", "em"],
+        "mountPoint": []
+    },
+    "default_tags": {
     },
     "ignore": {
         "cpu.busy": true,
```

###2. 升级transfer
用falcon-plus transfer 二进制文件替换掉v0.1版本 transfer 的二进制即可，配置文件无需变化。

###3. 升级graph
用falcon-plus graph 二进制文件替换掉v0.1版本 graph 的二进制即可，配置文件无需变化。

###4. 用api模块来代替query模块
  1.  停止query的运行；
  2. 从 `make pack`得到的 tar.gz 包中解压得到 api 及其配置文件，检查配置文件如下，然后启动api组件即可。

```
{
        "log_level": "debug",
        "db": {
                "faclon_portal": "root:@tcp(127.0.0.1:3306)/falcon_portal?charset=utf8&parseTime=True&loc=Local",
                "graph": "root:@tcp(127.0.0.1:3306)/graph?charset=utf8&parseTime=True&loc=Local",
                "uic": "root:@tcp(127.0.0.1:3306)/uic?charset=utf8&parseTime=True&loc=Local",
                "dashboard": "root:@tcp(127.0.0.1:3306)/dashboard?charset=utf8&parseTime=True&loc=Local",
                "alarms": "root:@tcp(127.0.0.1:3306)/alarms?charset=utf8&parseTime=True&loc=Local",
                "db_bug": true
        },
        "graphs": {
                "cluster": {
                        "graph-00": "127.0.0.1:6070"
                },
                "max_conns": 100,
                "max_idle": 100,
                "conn_timeout": 1000,
                "call_timeout": 5000,
                "numberOfReplicas": 500
        },
        "metric_list_file": "./api/data/metric",
        "web_port": ":8080",
        "access_control": true,
        "salt": "",
        "skip_auth": false,
        "signup_disable": false,
        "default_token": "default-token-used-in-server-side",
        "gen_doc": false,
        "gen_doc_path": "doc/module.html"
}

```
 其中`db`部分为数据库连接信息相关，按需修改即可；`graphs`部分为后端的graph列表信息，可以和v0.1版本的query cfg.json部分内容保持一致即可。

###5. 升级hbs
用falcon-plus hbs 二进制文件替换掉v0.1版本 hbs 的二进制即可，配置文件无需变化。

###6. 升级judge
用falcon-plus judge 二进制文件替换掉v0.1版本 judge 的二进制即可，配置文件无需变化。

###7. 升级alarm
用falcon-plus alarm 二进制文件替换掉v0.1版本 alarm 的二进制，同时注意修改alarm 的配置文件：

```
diff --git v0.1/alarm/cfg.example.json v0.2/alarm/cfg.example.json
--- v0.1/alarm/cfg.example.json
+++ v0.2/alarm/cfg.example.json

@@ -1,36 +1,47 @@
 {
-    "debug": true,
-    "uicToken": "",
+    "log_level": "debug",
     "http": {
         "enabled": true,
         "listen": "0.0.0.0:9912"
     },
-    "queue": {
-        "sms": "/sms",
-        "mail": "/mail"
-    },
     "redis": {
         "addr": "127.0.0.1:6379",
         "maxIdle": 5,
@@@@@@@@@@
+        "userIMQueue": "/queue/user/im",
         "userSmsQueue": "/queue/user/sms",
         "userMailQueue": "/queue/user/mail"
     },
-       "database": "root:@tcp(127.0.0.1:3306)/alarm?loc=Local&parseTime=true",
-       "maxIdle": 100,
     "api": {
-        "portal": "http://falcon.example.com",
-        "uic": "http://uic.example.com",
-        "links": "http://link.example.com"
+        "im": "http://127.0.0.1:10086/wechat",
+        "sms": "http://127.0.0.1:10086/sms",
+        "mail": "http://127.0.0.1:10086/mail",
+        "dashboard": "http://127.0.0.1:8081",
+        "plus_api":"http://127.0.0.1:8080",
+        "plus_api_token": "default-token-used-in-server-side"
+    },
+    "falcon_portal": {
+        "addr": "root:@tcp(127.0.0.1:3306)/alarms?charset=utf8&loc=Asia%2FChongqing",
+        "idle": 10,
+        "max": 100
+    },
+    "worker": {
+        "im": 10,
+        "sms": 10,
+        "mail": 50
+    },
+    "housekeeper": {
+        "event_retention_days": 7,
+        "event_delete_batch": 100
     }
 }
```
*NOTE: v0.2版本中，去除了sender模块，将sender模块的功能，合并到了alarm组件中，降低了用户的配置和维护成本; 同时增加了对微信的发送支持。*

###8. 升级aggregator
用falcon-plus aggregator 二进制文件替换掉v0.1版本 aggregator 的二进制，同时注意修改aggregator 的配置文件：

```
diff --git v0.1/aggregator/cfg.example.json v0.2/aggregator/cfg.example.json
--- v0.1/aggregator/cfg.example.json
+++ v0.2/aggregator/cfg.example.json
@@@@@@@@@@@
         "interval": 55
     },
     "api": {
-        "hostnames": "http://127.0.0.1:5050/api/group/%s/hosts.json",
-        "push": "http://127.0.0.1:6060/api/push",
-        "graphLast": "http://127.0.0.1:9966/graph/last"
+        "connect_timeout": 500,
+        "request_timeout": 2000,
+        "plus_api": "http://127.0.0.1:8080",
+        "plus_api_token": "default-token-used-in-server-side",
+        "push_api": "http://127.0.0.1:1988/v1/push"
     }
 }
```
*NOTE: aggregator 直接和falcon-plus 的api组件进行交互，即plus_api所配置的地址。*

###9. 升级nodata
用falcon-plus nodata 二进制文件替换掉v0.1版本 nodata 的二进制，同时注意修改nodata 的配置文件：

```
diff --git v0.1/nodata/cfg.example.json v0.2/nodata/cfg.example.json
--- a/../nodata/cfg.example.json
+++ b/modules/nodata/cfg.example.json
@@ -4,10 +4,11 @@
         "enabled": true,
         "listen": "0.0.0.0:6090"
     },
-    "query":{
-        "connectTimeout": 5000,
-        "requestTimeout": 30000,
-        "queryAddr": "127.0.0.1:9966"
+    "plus_api":{
+        "connectTimeout": 500,
+        "requestTimeout": 2000,
+        "addr": "http://127.0.0.1:8080",
+        "token": "default-token-used-in-server-side"
     },
     "config": {
         "enabled": true,
@@ -21,13 +22,9 @@
     },
     "sender":{
         "enabled": true,
-        "connectTimeout": 5000,
-        "requestTimeout": 30000,
+        "connectTimeout": 500,
+        "requestTimeout": 2000,
         "transferAddr": "127.0.0.1:6060",
-        "batch": 500,
-        "block": {
-            "enabled": false,
-            "threshold": 32
-        }
+        "batch": 500
     }
 }
```
*NOTE: nodata不再访问query组件了， 而是直接和falcon-plus 的api组件进行交互，即plus_api所配置的信息。*

###10. 升级gateway
用falcon-plus gateway 二进制文件替换掉v0.1版本 gateway 的二进制即可，配置文件无需变化。

###11. 开启全新的统一[前端模块](https://github.com/open-falcon/dashboard)
- 安装步骤请直接参考：[dashboard readme](https://github.com/open-falcon/dashboard) 
- 注意修改配置：

```
diff --git a/rrd/config.py b/rrd/config.py
index e9e500d..d57b1c7 100755
--- a/rrd/config.py
+++ b/rrd/config.py
@@ -1,34 +1,50 @@
 #-*-coding:utf8-*-
 import os
+LOG_LEVEL = os.environ.get("LOG_LEVEL",'DEBUG')
+SECRET_KEY = os.environ.get("SECRET_KEY","secret-key")
+PERMANENT_SESSION_LIFETIME = os.environ.get("PERMANENT_SESSION_LIFETIME",3600 * 24 * 30)
+SITE_COOKIE = os.environ.get("SITE_COOKIE","open-falcon-ck")
 
-#-- dashboard db config --
-DASHBOARD_DB_HOST = "127.0.0.1"
-DASHBOARD_DB_PORT = 3306
-DASHBOARD_DB_USER = "root"
-DASHBOARD_DB_PASSWD = ""
-DASHBOARD_DB_NAME = "dashboard"
+# Falcon+ API
+API_ADDR = os.environ.get("API_ADDR","http://127.0.0.1:8080/api/v1")
 
-#-- graph db config --
-GRAPH_DB_HOST = "127.0.0.1"
-GRAPH_DB_PORT = 3306
-GRAPH_DB_USER = "root"
-GRAPH_DB_PASSWD = ""
-GRAPH_DB_NAME = "graph"
+# portal database
+# TODO: read from api instead of db
+PORTAL_DB_HOST = os.environ.get("PORTAL_DB_HOST","127.0.0.1")
+PORTAL_DB_PORT = int(os.environ.get("PORTAL_DB_PORT",3306))
+PORTAL_DB_USER = os.environ.get("PORTAL_DB_USER","root")
+PORTAL_DB_PASS = os.environ.get("PORTAL_DB_PASS","")
+PORTAL_DB_NAME = os.environ.get("PORTAL_DB_NAME","falcon_portal")
 
-#-- app config --
-DEBUG = True
-SECRET_KEY = "secret-key"
-SESSION_COOKIE_NAME = "open-falcon"
-PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30
-SITE_COOKIE = "open-falcon-ck"
+# alarm database
+# TODO: read from api instead of db
+ALARM_DB_HOST = os.environ.get("ALARM_DB_HOST","127.0.0.1")
+ALARM_DB_PORT = int(os.environ.get("ALARM_DB_PORT",3306))
+ALARM_DB_USER = os.environ.get("ALARM_DB_USER","root")
+ALARM_DB_PASS = os.environ.get("ALARM_DB_PASS","")
+ALARM_DB_NAME = os.environ.get("ALARM_DB_NAME","alarms")
 
-#-- query config --
-QUERY_ADDR = "http://127.0.0.1:9966"
+# ldap config
+LDAP_ENABLED = os.environ.get("LDAP_ENABLED",False)
+LDAP_SERVER = os.environ.get("LDAP_SERVER","ldap.forumsys.com:389")
+LDAP_BASE_DN = os.environ.get("LDAP_BASE_DN","dc=example,dc=com")
+LDAP_BINDDN_FMT = os.environ.get("LDAP_BINDDN_FMT","uid=%s,dc=example,dc=com")
+LDAP_SEARCH_FMT = os.environ.get("LDAP_SEARCH_FMT","uid=%s")
+LDAP_ATTRS = ["cn","mail","telephoneNumber"]
+LDAP_TLS_START_TLS = False
+LDAP_TLS_CACERTDIR = ""
+LDAP_TLS_CACERTFILE = "/etc/openldap/certs/ca.crt"
+LDAP_TLS_CERTFILE = ""
+LDAP_TLS_KEYFILE = ""
+LDAP_TLS_REQUIRE_CERT = True
+LDAP_TLS_CIPHER_SUITE = ""
 
-BASE_DIR = "/home/work/open-falcon/dashboard/"
-LOG_PATH = os.path.join(BASE_DIR,"log/")
+# portal site config
+MAINTAINERS = ['root']
+CONTACT = 'root@open-falcon.com'
```
