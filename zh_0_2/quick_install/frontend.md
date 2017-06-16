## 环境准备

请参考[环境准备](./prepare.md)

### 创建工作目录
```
export HOME=/home/work
export WORKSPACE=$HOME/open-falcon
mkdir -p $WORKSPACE
cd $WORKSPACE
```

### 克隆前端组件代码
```
cd $WORKSPACE
git clone https://github.com/open-falcon/dashboard.git
```

### 安装依赖包
```
yum install -y python-virtualenv
yum install -y python-devel
yum install -y openldap-devel
yum install -y mysql-devel
yum groupinstall "Development tools"


cd $WORKSPACE/dashboard/
virtualenv ./env

./env/bin/pip install -r pip_requirements.txt
```

### 初始化数据库
请参考[环境准备](./prepare.md)


### 修改配置
```
dashboard的配置文件为： 'rrd/config.py'，请根据实际情况修改

## API_ADDR 表示后端api组件的地址
API_ADDR = "http://127.0.0.1:8080/api/v1" 

## 根据实际情况，修改PORTAL_DB_*, 默认用户名为root，默认密码为""
## 根据实际情况，修改ALARM_DB_*, 默认用户名为root，默认密码为""
```

### 以开发者模式启动
```
./env/bin/python wsgi.py

open http://127.0.0.1:8081 in your browser.
```

### 在生产环境启动
```
bash control start

open http://127.0.0.1:8081 in your browser.
```

### 停止dashboard运行
```
bash control stop
```

### 查看日志
```
bash control tail
```

