# Fe

这是Go版本的[UIC](http://ulricqin.com/project/uic/)，也是一个统一的web入口，因为监控组件众多，记忆ip、port去访问还是比较麻烦。fe像是一个监控的hao123

## 设计初衷

一个软件的部署分发难易程度，直接影响了软件的流传。社区有些朋友反映Java版的UIC难以安装，我们就提供了一个Go版本。大家可以在Fe中维护个人联系信息，维护人和组的对应关系。还可以通过配置文件中配置的shortcut在页面上产生多个快捷方式链接。

## 与UIC区别

Fe模块除了提供了一个简单的导航之外，最大的不同是密码存放方式发生了变化，所以Java版UIC用户如果要迁移过来，需要修改Fe模块配置的salt，配置为空字符串，就可以和原来Java版本的UIC共用同一个数据库了，不过配置成空字符串不够安全，建议salt配置一个随机字符串，然后通过Fe注册一个新用户，把数据库中所有用户的密码都重置为这个新用户的密码，发个通知，让各个注册用户重新自己登录修改密码。

## 源码安装

```bash
cd $GOPATH/src/github.com/open-falcon/fe
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的包，拿着这个包去部署即可。

## 部署说明

Fe作为一个前端模块，无状态，可以水平扩展，至少部署两台机器以保证可用性。前面做一个负载均衡设备，nginx或者lvs都可以。最后为其申请一个域名，搞定！

## 配置介绍

配置文件必须叫cfg.json，可以基于cfg.example.json修改

```
{
    "log": "debug",
    "company": "MI", # 填写自己公司的名称，用于生成联系人二维码
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:1234" # 自己随便搞个端口，别跟现有的重复了，可以使用8080，与老版本保持一致
    },
    "cache": {
        "enabled": true,
        "redis": "127.0.0.1:6379", # 这个redis跟judge、alarm用的redis不同，这个只是作为缓存来用
        "idle": 10,
        "max": 1000,
        "timeout": {
            "conn": 10000,
            "read": 5000,
            "write": 5000
        }
    },
    "salt": "0i923fejfd3", # 搞一个随机字符串
    "canRegister": true,
    "ldap": {
        "enabled": false,
        "addr": "ldap.example.com:389",
        "baseDN": "dc=example,dc=com",
        "bindDN": "cn=mananger,dc=example,dc=com", 
        "bindPasswd": "12345678",
        "userField": "uid",
        "attributes": ["sn","mail","telephoneNumber"] 
    },
    "uic": {
        "addr": "root:password@tcp(127.0.0.1:3306)/fe?charset=utf8&loc=Asia%2FChongqing",
        "idle": 10,
        "max": 100
    },
    "shortcut": {
        "falconPortal": "http://11.11.11.11:5050/", 浏览器可访问的portal地址
        "falconDashboard": "http://11.11.11.11:7070/", 浏览器可访问的dashboard地址
        "falconAlarm": "http://11.11.11.11:6060/" 浏览器可访问的alarm的http地址
    }
}
```

## 进程管理

我们提供了一个control脚本来完成常用操作

```bash
./control start 启动进程
./control stop 停止进程
./control restart 重启进程
./control status 查看进程状态
./control tail 用tail -f的方式查看var/app.log
```

## 设置root账号的密码

该项目中的注册用户是有不同角色的，目前分三种角色：普通用户、管理员、root账号。系统启动之后第一件事情应该是设置root的密码，浏览器访问：http://fe.example.com/root?password=abc （此处假设你的项目访问地址是fe.example.com，也可以使用ip）,这样就设置了root账号的密码为abc。普通用户可以支持注册。

## ldap 认证
Fe 现在支持通过 ldap 来进行用户认证。不需要提前在 Fe 内开设账号。Fe 会自动将 ldap 认证过来的新用户插入到 Fe 的数据库内。

配置说明
```
        "addr": "ldap.example.com:389",
		# ldap 的地址和端口

        "baseDN": "dc=example,dc=com",
	    # ldap 的 baseDN，ldap 认证的时候将从这个路径开始查询用户

        "bindDN": "cn=mananger,dc=example,dc=com", 
		# 你用来连接 ldap 的账户，至少要有只读的查询权限。
		# 注意这里应该是账户的完整 dn 值。对于 AD 的话，则可以直接填账户的 userPrincipalName (xxx@example.com）。
		# 如果你的 ldap 允许匿名查询的话，填""值即可

        "bindPasswd": "12345678", 
		# 如果你的 ldap 允许匿名查询的话，填""值即可

        "userField": "uid", 
		# 用于认证的属性（即你输入的用户名），通常为 uid 或 sAMAccountName(AD)。
		# 也可以使用诸如 mail 的属性，这样认证的用户名就是邮箱(前提 ldap 里有这个属性)

        "attributes": ["sn","mail","telephoneNumber"] 
		# 数组顺序重要，依次为姓名，邮箱，电话在 ldap 中的属性名.
		# 建议根据自己 ldap 内的实际情况修改。
		# 用户 ldap 登录时，fe 将按这些属性名去 ldap 中查询新用户的属性，并插入到 fe 的数据库内。
    },
```


## 补充

这里我们先安装了Fe这个模块，portal、dashboard、alarm等模块都还没有安装，所以shortcut中不知道如何配置才好。不用着急，先维持默认，等之后部署完了portal、dashboard、alarm等模块之后再回来修改fe的配置。

## 视频教程

为fe模块录制了一个视频，做了源码级解读：http://www.jikexueyuan.com/course/1780.html

