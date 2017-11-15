<!-- toc -->

# Agent-updater

每台机器都要部署falcon-agent，如果公司机器量比较少，用pssh、ansible、fabric之类的工具手工安装问题也不大。但是公司机器量多了之后，手工安装、升级、回滚falcon-agent将成为噩梦。

个人开发了agent-updater这个工具，用于管理falcon-agent，agent-updater也有一个agent：ops-updater，可以看做是一个超级agent，用于管理其他agent的agent，呵呵，ops-updater推荐在装机的时候一起安装上。ops-updater通常是不会升级的。

具体参看：https://github.com/open-falcon/ops-updater 

如果你想学习如何使用Go语言编写一个完整的项目，也可以研究一下agent-updater，我甚至录制了一个视频教程来演示一步一步如何开发出来的。课程链接：

- http://www.jikexueyuan.com/course/1336.html
- http://www.jikexueyuan.com/course/1357.html
- http://www.jikexueyuan.com/course/1462.html
- http://www.jikexueyuan.com/course/1490.html

