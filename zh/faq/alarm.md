# 报警相关常见问题

#### 配置了策略，一直没有报警，如何排查？

1. 排查sender、alarm、judge、hbs、agent、transfer的log
2. 浏览器访问alarm的http页面，看是否有未恢复的告警，如果有就是生成报警了，后面没发出去，很可能是邮件、短信发送接口出问题了
3. 打开agent的debug，看是否在正常push数据
4. 看agent配置，是否正确配置了heartbeat(hbs)和transfer的地址，并enabled
5. 看transfer配置，是否正确配置了judge地址
6. jduge提供了一个http接口用于debug，可以检查某个数据是否正确push上来了，比如qd-open-falcon-judge01.hd这个机器的cpu.idle数据，可以这么查看

```bash
curl 127.0.0.1:6081/history/qd-open-falcon-judge01.hd/cpu.idle
```

上面的127.0.0.1:6081指的是judge的http端口

7. 检查judge配置的hbs地址是否正确
8. 检查hbs配置的数据库地址是否正确
9. 检查portal中配置的策略模板是否配置了报警接收人
10. 检查portal中配置的策略模板是否绑定到某个HostGroup了，并且目标机器恰好在这个HostGroup中
11. 去UIC检查报警接收组中是否把自己加进去了
12. 去UIC检查自己的联系信息是否正确

