# 邮件、短信、微信发送接口

这个组件没有代码，需要各个公司自行提供。

监控系统产生报警事件之后需要发送报警邮件或者报警短信，各个公司可能有自己的邮件服务器，有自己的邮件发送方法；有自己的短信通道，有自己的短信发送方法。falcon为了适配各个公司，在接入方案上做了一个规范，需要各公司提供http的短信和邮件发送接口

短信发送http接口：

```
method: post
params:
  - content: 短信内容
  - tos: 使用逗号分隔的多个手机号
```

邮件发送http接口：

```
method: post
params:
  - content: 邮件内容
  - subject: 邮件标题
  - tos: 使用逗号分隔的多个邮件地址
```


im发送http接口：

```
method: post
params:
  - content: im内容
  - tos: 使用逗号分隔的多个im号码
```

不过你可以使用社区提供的`邮件发送网关`和`微信网关`

- [邮件网关](https://github.com/open-falcon/mail-provider)
- [微信网关](https://github.com/Yanjunhui/chat)
