##Links

Links is a component written for alarm merging function. If you don't want to use alarm merging function, there is no need to install this component.

##Source code installation

Links is a Python project, and there is no need to compile like a project of Go. However, a Go project is statically compiled. There is no binary dependency after compiling, so it can run in other devices, while some dependency libraries are needed for a Python project.

```
# We use virtualenv to manage Python environment, and it is necessary to switch to the root account to install yum

# yum install -y python-virtualenv

$ cd /path/to/links/
$ virtualenv ./env

$ ./env/bin/pip install -r pip_requirements.txt
```

After installing the dependent libraries, we can use the control script to start it, and logs will be located at the var directory. But it is necessary to modify the configuration file into the corresponding configuration before starting. Besides, the monitoring port should be configured in gunicorn.conf.

##Deployment instruction
Links is a stateless web project, which can be scaled horizontally, and need at least two computers to ensure the availability. Set up a nginx or lvs loading equipment in the front and apply for a domain name, that's all!

##Configuration instruction

The configuration files of Links are located at frame/config.py.

```
# Modify the database configuration, and database schema files are located at the Scripts directory.
DB_HOST = "127.0.0.1"
DB_PORT = 3306
DB_USER = "root"
DB_PASS = ""
DB_NAME = "falcon_links"

# SECRET_KEY	Set a complicated random character string串
SECRET_KEY = "4e.5tyg8-u9ioj"
SESSION_COOKIE_NAME = "falcon-links"
PERMANENT_SESSION_LIFETIME = 3600 * 24 * 30

# We can copy config.py local_config.py, and use the configuration in local_config.py to overwrite the configuration in config.py
#If this is inconvenient, you can just maintain the default configuration, and there is no need to make local_config.py
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
./control status	View the process state
./control tail	Use the method of "tail -f" to view var/app.log
```
##Verification
Check whether the log located in the var directory is normal after start.

Then use the browser to visit it, if the home page appears 404, it is normal. The alarm module will use links.

Or we can verify as follows:

```curl http://links.example.com/store -d"abc"```

The above command will return a random character string, then we can add the string at the end of the links address to visit the new address through a browser. For example, if the returned random character string is dot9kg8b, then use the browser to visit: http://links.example.com/dot9kg8b.