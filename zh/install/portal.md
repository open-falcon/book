# Portal

Portal是用来配置报警策略的

## 源码安装

Portal是个Python的项目，无需像Go的项目那样去做编译。不过Go的项目是静态编译的，编译好了之后二进制无依赖，拿到其他机器也可以跑起来，Python的项目就需要安装一些依赖库了。

```bash
# 我们使用virtualenv来管理Python环境，yum安装需切到root账号
# yum install -y python-virtualenv

$ cd /path/to/portal/
$ virtualenv ./env

# use douban pypi
$ ./env/bin/pip install -r pip_requirements.txt -i http://pypi.douban.com/simple
```

安装完依赖的lib之后就可以用control脚本启动了，log在var目录。不过启动之前要先把配置文件修改成相应配置。另外，监听的端口在gunicorn.conf中配置。


## 部署说明

Portal是个web项目，无状态，可以水平扩展，至少部署两台机器以保证可用性，前面架设nginx或者lvs这种负载设备，申请一个域名，搞定！

## 配置说明

Portal的配置文件在frame/config.py

```python
# 修改一下数据库配置，数据库schema文件在scripts目录
DB_HOST = "127.0.0.1"
DB_PORT = 3306
DB_USER = "root"
DB_PASS = ""
DB_NAME = "falcon_portal"

# SECRET_KEY尽量搞一个复杂点的随机字符串
SECRET_KEY = "4e.5tyg8-u9ioj"
SESSION_COOKIE_NAME = "falcon-portal"
PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30

# 如果你使用的是Go版本的UIC，即Fe那个项目，下面的配置就配置成Fe的地址即可，注意端口，Fe的默认端口是1234
# internal是内网可访问的UIC（或者Fe）地址
# external是外网可访问的UIC（或者Fe）地址，即用户通过浏览器访问的UIC（或者Fe）地址
UIC_ADDRESS = {
    'internal': 'http://127.0.0.1:8080',
    'external': 'http://11.11.11.11:8080',
}

MAINTAINERS = ['root']
CONTACT = 'ulric.qin@gmail.com'

# 社区版必须维持默认配置
COMMUNITY = True

# 我们可以cp config.py local_config.py用local_config.py中的配置覆盖config.py中的配置
# 嫌麻烦的话维持默认即可，也不用制作local_config.py
try:
    from frame.local_config import *
except Exception, e:
    print "[warning] %s" % e
```

# 补充

Portal正常启动之后，就可以回去配置Fe这个项目的shortcut了。当然，dashboard和alarm还没有搭建，这俩shortcut还没法配置。修改完了shortcut，要重启fe模块。
