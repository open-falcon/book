## 环境准备

请参考[环境准备](./prepare.md)

同时，请再次检查当前的工作目录设置：

	export HOME=/home/work
	export WORKSPACE=$HOME/open-falcon
	mkdir -p $WORKSPACE

# mail-provider & sms-provider

监控系统产生报警事件之后需要发送报警邮件或者报警短信，各个公司可能有自己的邮件服务器，有自己的邮件发送方法；有自己的短信通道，有自己的短信发送方法。falcon为了适配各个公司，在接入方案上做了一个规范，需要各公司提供http的短信和邮件发送接口。

短信发送http接口：

```bash
method: post
params:
  - content: 短信内容
  - tos: 使用逗号分隔的多个手机号
```

falcon将这样调用该接口：

```bash
url=您公司提供的http短信接口
curl -X POST $url -d "content=xxx&tos=18611112222,18611112223"
```

邮件发送http接口：

```bash
method: post
params:
  - content: 邮件内容
  - subject: 邮件标题
  - tos: 使用逗号分隔的多个邮件地址
```

falcon将这样调用该接口：

```bash
url=您公司提供的http邮件接口
curl -X POST $url -d "content=xxx&tos=ulric.qin@gmail.com,user@example.com&subject=xxx"
```

# sender

调用各个公司提供的mail-provider和sms-provider，按照某个并发度，从redis中读取邮件、短信并发送，alarm生成的报警短信和报警邮件都是直接写入redis即可，sender来发送。

```bash
cd $WORKSPACE/sender/
mv cfg.example.json cfg.json
# vi cfg.json
# redis地址需要和后面的alarm、judge使用同一个
# queue维持默认
# worker是最多同时有多少个线程玩命得调用短信、邮件发送接口
# api要给出sms-provider和mail-provider的接口地址
./control start
```

# fe

```bash
cd $WORKSPACE/fe/
mv cfg.example.json cfg.json
# 请基于cfg.example.json 酌情修改相关配置项

# 启动
./control start

# 查看日志
./control tail

# 停止服务
./control stop

```
# portal

portal是用于配置报警策略的地方

```bash
yum install -y python-virtualenv  # run as root

cd $WORKSPACE/portal/
virtualenv ./env

# use douban pypi
./env/bin/pip install -r pip_requirements.txt -i http://pypi.douban.com/simple

# vi frame/config.py
# 1. 修改DB配置
# 2. SECRET_KEY设置为一个随机字符串
# 3. UIC_ADDRESS有两个，internal配置为FE模块的内网地址，portal通常是和UIC在一个网段的，
#    内网地址相互访问速度快。external是终端用户通过浏览器访问的UIC地址，很重要！
# 4. 其他配置可以使用默认的

./control start

portal默认监听在5050端口，浏览器访问即可
```

# HBS（Heartbeat Server）
心跳服务器，只依赖Portal的DB

```bash
cd $WORKSPACE/hbs/
mv cfg.example.json cfg.json
# vi cfg.json 把数据库配置配置为portal的db
./control start
```

如果先安装的绘图组件又来安装报警组件，那应该已经安装过agent了，hbs启动之后会监听一个http端口，一个rpc端口，agent要和hbs通信，重新去修改agent的配置cfg.json，把heartbeat那项enabled设置为true，并配置上hbs的rpc地址，`./control restart`重启agent，之后agent就可以和hbs心跳了

# judge
报警判断模块，judge依赖于HBS，所以得先搭建HBS

```bash
cd $WORKSPACE/judge/
mv cfg.example.json cfg.json
# vi cfg.json
# remain: 这个配置指定了judge内存中针对某个数据存多少个点，比如host01这个机器的cpu.idle的值在内存中最多存多少个，
# 配置报警的时候比如all(#3)，这个#后面的数字不能超过remain-1
# hbs: 配置为hbs的地址，interval默认是60s，表示每隔60s从hbs拉取一次策略
# alarm: 报警event写入alarm中配置的redis，minInterval表示连续两个报警之间至少相隔的秒数，维持默认即可
./control start
```

# alarm
alarm模块是处理报警event的，judge产生的报警event写入redis，alarm从redis读取，这个模块被业务搞得很糟乱，各个公司可以根据自己公司的需求重写

```bash
cd $WORKSPACE/alarm/
mv cfg.example.json cfg.json
# vi cfg.json
# 把redis配置成与judge同一个

./control start
```

注意，alarm当前的版本，highQueues和lowQueues都不能为空，是个bug，稍候修复。我们可以把event:p0~event:p5配置到highQueues，把event:p6配置到lowQueues

# 告警合并功能
默认情况下，Open-Falcon未开启告警合并功能。如需开启该功能，请参考: [配置告警合并](./links.md)
