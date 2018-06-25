<!-- toc -->

## [urlooker][1]
Urlooker, which is written in Go language, monitors the availability of Web service and the visit quality and it is easy to install and redevelop.

## Feature
- Detect the state code of response
- Detect the response time of a page
- Matching detect of key word in a page
- Visit with cookie
- Agent deployment in multiple machine room; visit specific machine room
- Support sending detection result to Open-Falcon
- Support sending alarms via SMS and mail

## Architecture
![此处输入图片的描述][2]

## ScreenShot

![看图][3]

![此处输入图片的描述][4]

![添加监控项][5]

## Sent Metrics(details in [wiki][6])
- metric: url_status
- endpoint: url_id (ID in picture number 2)
- tag: creator=username (username in the top right of the picture 6)
- counterType: GAUGE
- step: 60 (can be adjusted in the configuration of web application)
- value: 0 (0~4 0 means normal, others mean innormal)

## Install

**Environment Dependency**   
Install mysql & redis      
wget http://x2know.qiniudn.com/schema.sql      
Import schema.sql to the database  

Binary installation(compiled in Ubuntu 14.4 Go1.6)：

    wget http://x2know.qiniudn.com/urlooker.tar.gz
    tar xzvf urlooker.tar.gz
    cd urlooker
    # Adjust the configuration of MySQL and redis in cfg.json
    web/control start
    alarm/control start
    agent/control start

Visit http://127.0.0.1:1984 with your browser.


## Help
Please refer to [urlooker][7] for source code installation and other detailed information. 


  [1]: https://github.com/urlooker
  [2]: https://github.com/urlooker/wiki/raw/master/img/urlooker4.png
  [3]: https://github.com/urlooker/wiki/raw/master/img/urlooker1.png
  [4]: https://github.com/urlooker/wiki/raw/master/img/urlooker3.png
  [5]: https://github.com/urlooker/wiki/raw/master/img/urlooker2.png
  [6]: https://github.com/URLooker/wiki/wiki/open-falcon-%E4%B8%8A%E6%8A%A5%E9%A1%B9%E4%BB%8B%E7%BB%8D
  [7]: https://github.com/710leo/urlooker/blob/master/README.md