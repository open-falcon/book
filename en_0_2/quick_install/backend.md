<!-- toc -->

## Environment Preparation

Please refer to [Environment Preparation](./prepare.md)

### Create a Working Directory
```bash
export FALCON_HOME=/home/work
export WORKSPACE=$FALCON_HOME/open-falcon
mkdir -p $WORKSPACE
```

### Unzip the Binary Pack
```bash
tar -xzvf open-falcon-v0.2.1.tar.gz -C $WORKSPACE
```

### Execute All the Backend Modules on One Machine

# First, make sure that the username and password in the configuration file of database are valid, or the configuration file should be edited.
```
cd $WORKSPACE
grep -Ilr 3306  ./ | xargs -n1 -- sed -i 's/root:/real_user:real_password/g'
```
# Execute
```bash
cd $WORKSPACE
./open-falcon start

# check the startup of all the modules
./open-falcon check

```

### More usage of command line tools
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
