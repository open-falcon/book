## 环境准备

请参考[环境准备](./prepare.md)

### 创建工作目录
```bash
export FALCON_HOME=/home/work
export WORKSPACE=$FALCON_HOME/open-falcon
mkdir -p $WORKSPACE
```

### 解压二进制包
```bash
tar -xzvf open-falcon-v0.2.0.tar.gz -C $WORKSPACE
```

### 在一台机器上启动所有的后端组件
# 首先确认配置文件中数据库账号密码与实际相同，否则需要修改配置文件。
```
cd $WORKSPACE
grep -Ilr 3306  ./ | xargs -n1 -- sed -i 's/root:password/real_user:real_password/g')
```
# 启动
```bash
cd $WORKSPACE
./open-falcon start

# 检查所有模块的启动状况
./open-falcon check

```

### 更多的命令行工具用法
```bash
# ./open-falcon [start|stop|restart|check|monitor|reload] module
./open-falcon start agent

./open-falcon check
        falcon-graph         UP           53007
          falcon-hbs         UP           53014
        falcon-judge         UP           53020
     falcon-transfer         UP           53026
       falcon-nodata         UP           53032
   falcon-aggregator         UP           53038
        falcon-agent         UP           53044
      falcon-gateway         UP           53050
          falcon-api         UP           53056
        falcon-alarm         UP           53063

For debugging , You can check $WorkDir/$moduleName/log/logs/xxx.log
```
