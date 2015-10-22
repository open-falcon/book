# Links

Links是为报警合并功能写的组件。如果你不想使用报警合并功能，这个组件是无需安装的。

## 源码安装

Links个Python的项目，无需像Go的项目那样去做编译。不过Go的项目是静态编译的，编译好了之后二进制无依赖，拿到其他机器也可以跑起来，Python的项目就需要安装一些依赖库了。

```bash
# 我们使用virtualenv来管理Python环境，yum安装需切到root账号
# yum install -y python-virtualenv

$ cd /path/to/links/
$ virtualenv ./env

# use douban pypi
$ ./env/bin/pip install -r pip_requirements.txt -i http://pypi.douban.com/simple
```

安装完依赖的lib之后就可以用control脚本启动了，log在var目录。不过启动之前要先把配置文件修改成相应配置。另外，监听的端口在gunicorn.conf中配置。


## 部署说明

Links是个web项目，无状态，可以水平扩展，至少部署两台机器以保证可用性，前面架设nginx或者lvs这种负载设备，申请一个域名，搞定！

## 配置说明

Links的配置文件在frame/config.py

```python
# 修改一下数据库配置，数据库schema文件在scripts目录
DB_HOST = "127.0.0.1"
DB_PORT = 3306
DB_USER = "root"
DB_PASS = ""
DB_NAME = "falcon_links"

# SECRET_KEY尽量搞一个复杂点的随机字符串
SECRET_KEY = "4e.5tyg8-u9ioj"
SESSION_COOKIE_NAME = "falcon-links"
PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30

# 我们可以cp config.py local_config.py用local_config.py中的配置覆盖config.py中的配置
# 嫌麻烦的话维持默认即可，也不用制作local_config.py
try:
    from frame.local_config import *
except Exception, e:
    print "[warning] %s" % e
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

## 验证

启动之后要看看log是否正常，log在var目录。

然后浏览器访问之，发现首页404，这是正常的。之后alarm模块会用到links。

或者我们可以这么验证：

```bash
curl http://links.example.com/store -d "abc"
```

上面命令会返回一个随机字符串，拿着这个随机字符串拼接到links地址后面，浏览器访问之即可。比如返回的随机字符串是dot9kg8b，浏览器访问：http://links.example.com/dot9kg8b 即可

