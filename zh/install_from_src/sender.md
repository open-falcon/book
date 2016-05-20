# Sender

上节我们利用http接口规范屏蔽了邮件、短信发送的问题。Sender这个模块专门用于调用各公司提供的邮件、短信发送接口。

## 设计初衷

各个公司会提供邮件、短信发送接口，我们产生了报警之后就立马调用这些接口发送报警，是不合适的。因为这些接口可能无法处理巨大的并发量，而且接口本身的处理速度可能也比较慢，这会拖慢我们的处理逻辑。所以一个比较好的方式是把邮件、短信发送这个事情做成异步的。

我们提供一个短信redis队列，提供一个邮件redis队列。当有短信要发送的时候，直接将短信内容写入短信redis队列即可，当有邮件要发送的时候，直接将邮件内容写入邮件redis队列。针对每个队列，后面有一个预设大小的worker线程池来处理。

有了队列的缓冲，即便某个时刻产生了大量报警，造成邮件、短信发送的突发流量，也不会对邮件、短信发送接口造成冲击。

## 源码安装

```bash
cd $GOPATH/src/github.com/open-falcon/sender
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的包，拿着这个包去部署即可。

## 部署说明

sender这个模块和redis队列部署在一台机器上即可。公司即使有几十万台机器，一个sender也足够了。

## 配置说明

配置文件必须叫cfg.json，可以基于cfg.example.json修改

```
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6066"
    },
    "redis": {
        "addr": "127.0.0.1:6379", # 此处配置的redis地址要和后面的judge、alarm配置成相同的
        "maxIdle": 5
    },
    "queue": {
        "sms": "/sms", # 短信队列名称，维持默认即可，alarm中也会有一个相同的配置
        "mail": "/mail" # 邮件队列名称，维持默认即可，alarm中也会有一个相同的配置
    },
    "worker": {
        "sms": 10, # 调用短信接口的最大并发量
        "mail": 50 # 调用邮件接口的最大并发量
    },
    "api": {
        "sms": "http://11.11.11.11:8000/sms", # 各公司自行提供的短信发送接口，11.11.11.11这个ip只是个例子喽
        "mail": "http://11.11.11.11:9000/mail" # 各公司自行提供的邮件发送接口
    }
}
```

如果没有邮件发送接口，可以使用 [Open-Falcon mail-provider](https://github.com/open-falcon/mail-provider)。

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

sender的配置文件中配置了监听的http端口，我们可以访问一下/health接口看是否返回ok，我们所有的Go后端模块都提供了/health接口，上面的配置的话就是这样验证：

```bash
curl 127.0.0.1:6066/health
```

另外就是查看sender的log，log在var目录下

## 视频教程

为sender模块录制了一个视频，做了源码级解读：http://www.jikexueyuan.com/course/1641.html

