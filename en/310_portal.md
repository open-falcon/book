##Portal

Portal is used to configure alarm strategies.

##Source code installation

Portal is a Python project, and there is no need to compile like a Go project. However, a Go project is statically compiled. There is no binary dependency after compiling, so it can run in other devices, while some dependency libraries are needed for a Python project.

```
# 我们使用virtualenv来管理Python环境，yum安装需切到root账号
# yum install -y python-virtualenv

$ cd /path/to/portal/
$ virtualenv ./env

$ ./env/bin/pip install -r pip_requirements.txt
```
After installing the dependent libraries, we can use the control script to start it, and logs will be located at the var directory. But it is necessary to modify the configuration file into the corresponding configuration before starting. Besides, the monitoring port should be configured in gunicorn.conf.

##Deployment instruction
Portal is a stateless Web project, which can be scaled horizontally, and need at least two computers to ensure the availability. Set up a nginx or lvs loading equipment in the front and apply for a domain name, that's all!

##Configuration instruction

The configuration files of Portal are located at frame/config.py.

```
# Modify the database configuration, and database schema files are located at the Scripts directory.
DB_HOST = "127.0.0.1"
DB_PORT = 3306
DB_USER = "root"
DB_PASS = ""
DB_NAME = "falcon_portal"

# SECRET_KEY	 Set a complicated random character string
SECRET_KEY = "4e.5tyg8-u9ioj"
SESSION_COOKIE_NAME = "falcon-portal"
PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30

# If you are using a Go version of UIC (i.e., the Fe project), configure the following configuration as the address of Fe. Pay attention to the port, the default port of Fe is 1234
# internal is a UIC (or Fe) address which can be visited through Intranet
# external is a UIC (or Fe) address which can be visited through Extranet, i.e., the UIC (or Fe) address visited by users through a browser.
UIC_ADDRESS = {
    'internal': 'http://127.0.0.1:8080',
    'external': 'http://11.11.11.11:8080',
}

MAINTAINERS = ['root']
CONTACT = 'ulric.qin@gmail.com'

# The default configuration must be maintained for the community version.
COMMUNITY = True

# We can copy config.py local_config.py, and use the configuration in local_config.py to overwrite the configuration in config.py
# If it is inconvenient for you, you can just maintain the default configuration, and there is no need to make local_config.py
try:
    from frame.local_config import *
except Exception, e:
    print "[warning] %s" % e
    
```

##Process management
We provide a control script to complete normal actions.
```
./control start	Start a process
./control stop	Stop a process
./control restart	Restart a process
./control status	View the process status
./control tail	Use the method of "tail -f" to view var/app.log
```
##Supplement
The shortcut of the Fe project can be configured after Portal starts normally. And the shortcuts of "dashboard" and "alarm" can not be configured because "dashboard" and "alarm" have not been set up. It is necessary to restart the fe module after modifying the shortcut.
##Video course
We provide a video recorded for this module to provide interpretation at source code level: http://www.jikexueyuan.com/course/1796.html
