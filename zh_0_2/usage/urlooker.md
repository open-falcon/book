## [urlooker](https://github.com/710leo/urlooker)
监控web服务可用性及访问质量，采用go语言编写，易于安装和二次开发    

## Feature
- 返回状态码检测
- 页面响应时间检测
- 页面关键词匹配检测
- 自定义Header
- GET、POST、PUT访问
- 自定义POST BODY
- 检测结果支持推送 open-falcon

## Architecture
![Architecture](https://github.com/710leo/urlooker/raw/master/img/urlooker_arch.png)

## ScreenShot

![ScreenShot](https://github.com/710leo/urlooker/blob/master/img/urlooker_en1.png)
![ScreenShot](https://github.com/710leo/urlooker/blob/master/img/urlooker_en2.png)
<img src="https://github.com/710leo/urlooker/blob/master/img/urlooker_stra.png" style="zoom:45%;" />

## 常见问题
- [wiki手册](https://github.com/710leo/urlooker/wiki)
- [常见问题](https://github.com/710leo/urlooker/wiki/FAQ)
- 初始用户名密码：admin/password

## Install
#### docker 安装

```bash
git clone https://github.com/710leo/urlooker.git
cd urlooker
docker build .
docker volume create urlooker-vol
# [CONTAINER ID] 在实际操作中需要替换为实际的镜像的ID
docker run -p 1984:1984 -d --name urlooker --mount source=urlooker-vol,target=/var/lib/mysql --restart=always [CONTAINER ID]
```

#### 源码安装

```bash
# 安装mysql
yum install -y mysql-server
wget https://raw.githubusercontent.com/710leo/urlooker/master/sql/schema.sql
mysql -h 127.0.0.1 -u root -p < schema.sql

# 安装组件
curl https://raw.githubusercontent.com/710leo/urlooker/master/install.sh|bash
cd $GOPATH/src/github.com/710leo/urlooker

# 将[mysql root password]替换为mysql root 数据库密码
sed -i 's/urlooker.pass/[mysql root password]/g' configs/web.yml

./control start all
```

打开浏览器访问 http://127.0.0.1:1984 即可

## 答疑
QQ群：556988374