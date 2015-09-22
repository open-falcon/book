开启告警合并功能，需要完成一下两个步骤：

## 调整alarm的配置
  
    cd $WORKSPACE/alarm/
  
    1. 将cfg.json中 highQueues 配置项的内容调整为
    [
      "event:p0",
      "event:p1"
    ] 
    2. 将cfg.json中 lowQueues 配置项的内容调整为
    [
      "event:p2",
      "event:p3",
      "event:p4",
      "event:p5",
      "event:p6"
    ] 

    说明：
    - 在Open-Falcon中，告警是分级别的，包括P0、P1 ... P6，告警优先级依次下降。
    - 对于高优先级的告警，Open-Falcon会保障优先发送。
    - 告警合并功能，只针对低优先级的告警生效，因为高优先级的告警一般都很重要，对实时性要求很高，不建议做告警合并。
    - 因此，在highQueues中配置的不会被合并，在lowQueues 中的会被合并，各位可以根据需求进行调整。

## 安装Links组件

links组件的作用：当多个告警被合并为一条告警信息时，短信中会附带一个告警详情的http链接地址，供用户查看详情。

### install dependency

    # yum install -y python-virtualenv
    $ cd $WORKSPACE/links/
    $ virtualenv ./env
    $ ./env/bin/pip install -r pip_requirements.txt -i http://pypi.douban.com/simple


### init database and config

    - database schema: https://github.com/open-falcon/scripts/blob/master/db_schema/links-db-schema.sql
    - database config: ./frame/config.py
    - 初始化Links的数据，也可以参考 [环境准备](https://github.com/open-falcon/doc/wiki/%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)有关Links的部分


### start

    $ cd $WORKSPACE/links/
    $ ./control start
    	--> goto http://127.0.0.1:5090

    $ ./control tail
    	--> tail log


