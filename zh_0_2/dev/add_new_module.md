<!-- toc -->

## 基于API的方式新增一个组件

首先实现该组件，然后修改API组件。假设新增组件叫NewModule，以下为修改API组件的具体步骤

#### 第一步， 添加路由并完成数据转发

- 在falcon-plus/modules/api/app/controller目录下新建NewModule目录，在该目录下添加路由Routes
- 使用utils.AuthSessionMidd作为中间件进行用户认证
- 实现组件的Forwarder，使用其作为每个路由的HandlerFunc，完成数据转发，或复用alarm_manager.Forwarder

具体实现方式可以参考：falcon-plus/modules/api/app/controller/alarm_manager

#### 第二步，注册路由

在falcon-plus/modules/api/app/controller/routes.go中添加NewModule.Routes注册新增路由

#### 第三步，添加NewModule地址配置

在API组件的配置文件cfg.json中添加NewModule的地址，比如："NewModule": "http://127.0.0.1:9922"

#### 第四步， 重新部署API组件
