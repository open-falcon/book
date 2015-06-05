# Task

task是监控系统一个必要的辅助模块。定时任务，实现了如下几个功能：

+ index更新。包括图表索引的全量更新 和 垃圾索引清理。
+ falcon服务组件的自身状态数据采集。定时任务了采集了transfer、graph、task这三个服务的内部状态数据。
+ falcon自检控任务。


## 源码编译

```bash
cd $GOPATH/src/github.com/open-falcon/task
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的安装包，拿着这个包去部署服务即可。

## 服务部署

服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```bash
# 修改配置, 配置项含义见下文
mv cfg.example.json cfg.json
vim cfg.json

# 启动服务
./control start

# 校验服务,这里假定服务开启了8001的http监听端口。检验结果为ok表明服务正常启动。
curl -s "127.0.0.1:8001/health"

...
# 停止服务
./control stop

```

服务启动后，可以通过日志查看服务的运行状态，日志文件地址为./var/app.log。可以通过调试脚本```./test/debug```查看服务器的内部状态数据，如 运行 ```bash ./test/debug``` 可以得到服务器内部状态的统计信息。


## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```
debug: true/false, 如果为true，日志中会打印debug信息

http
    - enable: true/false, 表示是否开启该http端口，该端口为控制端口，主要用来对task发送控制命令、统计命令、debug命令等
    - listen: 表示http-server监听的端口

index
    - enable: true/false, 表示是否开启索引更新任务
    - dsn: 索引服务的MySQL的连接信息，默认用户名是root，密码为空，host为127.0.0.1，database为graph（如有必要，请修改）
    - maxIdle: MySQL连接池配置，连接池允许的最大空闲连接数，保持默认即可
    - cluster: 后端graph列表，用具体的hostname:port表示

monitor
    - enable: true/false, 表示是否开启falcon的自监控任务
    - mailUrl: 邮件服务的http接口,用于发送自监控报警邮件
    - mainTos: 接收自监控报警邮件的邮箱地址,多个邮箱地址用逗号隔开
    - cluster: falcon后端服务列表，用具体的"module,hostname:port"表示，module取值可以为graph、transfer、judge、task等任意falcon组件

collector
    - enable: true/false, 表示是否开启falcon的自身状态采集任务
    - destUrl: 监控数据的push地址,默认为本机的1988接口
    - srcUrlFmt: 监控数据采集的url格式, %s将由机器名或域名替换
    - cluster: falcon后端服务列表，用具体的"module,hostname:port"表示，module取值可以为graph、transfer、task等       
```

## 补充说明
部署完成task组件后，请修改collector配置、使task能够正确采集transfer & graph的内部状态，请修改monitor配置、使task模块能够自检控Open-Falon的各组件(当前支持transfer、graph、query、judge等)。