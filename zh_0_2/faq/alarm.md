<!-- toc -->

# 报警相关常见问题

#### 配置了策略，一直没有报警，如何排查？

1. 排查sender、alarm、judge、hbs、agent、transfer的log
2. 浏览器访问alarm的http页面，看是否有未恢复的告警，如果有就是生成报警了，后面没发出去，很可能是邮件、短信发送接口出问题了，检查sender中配置的api
3. 打开agent的debug，看是否在正常push数据
4. 看agent配置，是否正确配置了heartbeat(hbs)和transfer的地址，并enabled
5. 看transfer配置，是否正确配置了judge地址
6. jduge提供了一个http接口用于debug，可以检查某个数据是否正确push上来了，比如qd-open-falcon-judge01.hd这个机器的cpu.idle数据，可以这么查看
```bash
curl http://127.0.0.1:6081/history/qd-open-falcon-judge01.hd/cpu.idle
```
7. 检查服务器的时间是否已经同步，可以用 [ntp](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Understanding_chrony_and-its_configuration.html) 或 chrony 来实现

上面的127.0.0.1:6081指的是judge的http端口
7. 检查judge配置的hbs地址是否正确
8. 检查hbs配置的数据库地址是否正确
9. 检查portal中配置的策略模板是否配置了报警接收人
10. 检查portal中配置的策略模板是否绑定到某个HostGroup了，并且目标机器恰好在这个HostGroup中
11. 去UIC检查报警接收组中是否把自己加进去了
12. 去UIC检查自己的联系信息是否正确

#### 在Portal页面创建了一个HostGroup，往HostGroup中增加机器的时候报错

1. 检查agent是否正确配置了heartbeat地址，并enabled了
2. 检查hbs log
3. 检查hbs配置的数据库地址是否正确
4. 检查hbs的配置hosts是否配置为sync了，只有留空的时候hbs才会去写host表，host表中有数据才能在页面上添加机器


#### 在alarm这边配置了短信、邮件、微信通知，告警写入都有，但实际发送有时候只有1种，有时2种，有时3种都有
1、检查是否有多个alarm进程同时读取一个redis队列，引起相互干扰，如urlooker的alarm。
2、修改redis队列名称，如修改urlooker的redis队列名称，使2个alarm读取不同的队列，避免造成干扰。



