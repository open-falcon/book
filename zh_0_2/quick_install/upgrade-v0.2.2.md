<!-- toc -->

> **本文将介绍：如何从falcon-plus v0.2.1 可以平滑升级到falcon-plus v0.2.2？**

**NOTE: 本次升级主要涉及到三部分！！**

###1. 增加alarm-manager模块
- 新增数据库表
```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/scripts/mysql/db_schema/
mysql -h 127.0.0.1 -u root -p < 6_alarm-manager-db-schema.sql
```

- 从 make pack得到的 tar.gz 包中解压得到 alarm-manager 及其配置文件，检查配置文件如下，然后启动alarm-manager组件即可。

```
{
    "log_level":"debug",
    "db":{
        "am": "root:@tcp(127.0.0.1:3306)/alarm_manager?charset=utf8&parseTime=True&loc=Local",
        "debug": true
    },
    "api": {
        "plus_api": "http://127.0.0.1:8080",
        "plus_api_token": "default-token-used-in-server-side"
    },
    "listen": ":9922"
}
```

###2. 升级api组件
- 用falcon-plus api 二进制替换掉v0.2.1版本的api二进制，同时修改配置文件。

```
diff --git v0.2.1/api/config/cfg.example.json  v0.2.2/api/config/cfg.example.json
--- v0.2.1/api/config/cfg.example.json
+++ v0.2.2/api/config/cfg.example.json
{
        "log_level": "debug",
        "db": {
                "falcon_portal": "root:@tcp(127.0.0.1:3306)/falcon_portal?charset=utf8&parseTime=True&loc=Local",
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
        "signup_disable": false,
        "salt": "",
        "skip_auth": false,
        "default_token": "default-token-used-in-server-side",
        "gen_doc": false,
-       "gen_doc_path": "doc/module.html"
+   "gen_doc_path": "doc/module.html",
+   "alarm_manager_api": "http://127.0.0.1:9922"
}
```

###3. 升级alarm组件
- 用falcon-plus alarm 二进制替换掉v0.2.1版本的alarm二进制，同时修改配置文件。

```
diff --git v0.2.1/alarm/cfg.example.json v0.2.2/alarm/cfg.example.json
--- v0.2.1/alarm/cfg.example.json
+++ v0.2.2/alarm/cfg.example.json
{
    "log_level": "debug",
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:9912"
    },
    "redis": {
        "addr": "127.0.0.1:6379",
        "maxIdle": 5,
        "highQueues": [
            "event:p0",
            "event:p1",
            "event:p2"
        ],
        "lowQueues": [
            "event:p3",
            "event:p4",
            "event:p5",
            "event:p6"
        ],
        "userIMQueue": "/queue/user/im",
        "userSmsQueue": "/queue/user/sms",
        "userMailQueue": "/queue/user/mail"
    },
    "api": {
        "im": "http://127.0.0.1:10086/wechat",
        "sms": "http://127.0.0.1:10086/sms",
        "mail": "http://127.0.0.1:10086/mail",
        "dashboard": "http://127.0.0.1:8081",
        "plus_api":"http://127.0.0.1:8080",
        "plus_api_token": "default-token-used-in-server-side"
    },
+    "alarm_channel": {
+        "enabled": false,
+        "alarm_manager_api": "http://127.0.0.1:9922/v1/recv"
+    },
    "falcon_portal": {
        "addr": "root:@tcp(127.0.0.1:3306)/alarms?charset=utf8&loc=Asia%2FChongqing",
        "idle": 10,
        "max": 100
    },
    "worker": {
        "im": 10,
        "sms": 10,
        "mail": 50
    },
    "housekeeper": {
+        "enabled": false,
        "event_retention_days": 7,
        "event_delete_batch": 100
    }
}

```
