<!-- toc -->

> **How to smoothly update OpenFalcon v0.1 to FalconPlus v0.2?**

### Add New Database List
```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/scripts/mysql/db_schema/
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
```

**NOTE:** 
1. If you want to know how to compile falcon-plus, please refer to [falcon-plus readme](https://github.com/open-falcon/falcon-plus) ;
2. Or download the official compiled [binary pack](https://github.com/open-falcon/falcon-plus/releases)；

---
Now let's begin our update.

###1. Update Agent

Replace the binary file of Agent in v0.1 with the binary file of Falcon-Plus Agent and make some modifications to the configuration file as follows：

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

###2. Update Transfer
Replace the binary file of Transfer in v0.1 with the binary file of Falcon-Plus Transfer. No need for modifications to the configuration file.

###3. Update Graph
Replace the binary file of Graph in v0.1 with the binary file of Falcon-Plus Graph. No need for modifications to the configuration file.

###4. Replace Query Module with API Module
  1. Stop Query from running
  2. Unzip API and its configuration file from the pack "tar.gz" required from `make pack`. Enable API module after checking the configuration file as follows:

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
`db` part is about the information of database connection. Just change it according to actual usage. `graphs` part is about the information of Graph list in the backend. Just keep its content the same as the content in cfg.json of Query in v0.1.

###5. Update HBS
Replace the binary file of HBS in v0.1 with the binary file of Falcon-Plus HBS. No need for modifications to the configuration file.

###6. Update Judge
Replace the binary file of Judge in v0.1 with the binary file of Falcon-Plus Judge. No need for modifications to the configuration file.

###7. Update Alarm
Replace the binary file of Alarm in v0.1 with the binary file of Falcon-Plus Alarm and make some modifications to the configuration file as follows：

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
*NOTE: Sender module was removed from v0.2 and its feature has been integrated to Alarm module, which lower the cost of user's configuration and maintenance. Sending via WeChat is supported by this version.*

###8. Update Aggregator
Replace the binary file of Aggregator in v0.1 with the binary file of Falcon-Plus Aggregator and make some modifications to the configuration file as follows：


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
*NOTE: Aggregator directly interacts  with the API of Falcon-Plus, which is the address configured by plus_api.*

###9. Update Nodata
Replace the binary file of Nodata in v0.1 with the binary file of Falcon-Plus Nodata and make some modifications to the configuration file as follows：

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
*NOTE: Nodata does not visit Query anymore. Instead, it directly communicate with the API of Falcon-Plus,  which is plus_api configuration.*

###10. Update Gateway
Replace the binary file of Gateway in v0.1 with the binary file of Falcon-Plus Gateway. No need for modifications to the configuration file.

###11. Enable Brand-New Integrated [Frontend Modules](https://github.com/open-falcon/dashboard)
- Please refer to [dashboard readme](https://github.com/open-falcon/dashboard) for installation instruction
- Please change the configuration as follows：

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
