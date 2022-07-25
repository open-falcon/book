<!-- toc -->

## [urlooker][1]
监控web服务可用性及访问质量，采用go语言编写，易于安装和二次开发

## Feature
- 返回状态码检测
- 页面响应时间检测
- 页面关键词匹配检测
- 带cookie访问
- agent多机房部署，指定机房访问
- 检测结果支持向open-falcon推送
- 支持短信和邮件告警

## Architecture
![此处输入图片的描述][2]

## ScreenShot

![看图][3]

![此处输入图片的描述][4]

![添加监控项][5]

## 上报项(详情见[wiki][6])
- metric: url_status
- endpoint: url_id  (id为上图2中的id)
- tag: creator=username（上图右上角的username）
- counterType: GAUGE
- step: 60（可在web组件配置文件设置）
- value: 0 (0~4 0表示正常，其他表示异常)

## Install

**环境依赖**   
安装mysql & redis      
wget http://x2know.qiniudn.com/schema.sql      
将schema.sql 导入数据库  

二进制安装(Ubuntu 14.4 Go1.6下编译)：

    wget http://x2know.qiniudn.com/urlooker.tar.gz
    tar xzvf urlooker.tar.gz
    cd urlooker
    # 修改下cfg.json中的mysql和redis配置
    web/control start
    alarm/control start
    agent/control start

打开浏览器访问 http://127.0.0.1:1984 即可


## 使用帮助
源码安装及详细介绍见：[urlooker][7]  


  [1]: https://github.com/urlooker
  [2]: https://github.com/urlooker/wiki/raw/master/img/urlooker4.png
  [3]: https://github.com/urlooker/wiki/raw/master/img/urlooker1.png
  [4]: https://github.com/urlooker/wiki/raw/master/img/urlooker3.png
  [5]: https://github.com/urlooker/wiki/raw/master/img/urlooker2.png
  [6]: https://github.com/URLooker/wiki/wiki/open-falcon-%E4%B8%8A%E6%8A%A5%E9%A1%B9%E4%BB%8B%E7%BB%8D
  [7]: https://github.com/710leo/urlooker/blob/master/README.md