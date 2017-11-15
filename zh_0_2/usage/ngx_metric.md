<!-- toc -->

# Nginx 监控

在[数据采集](../philosophy/data-collect.md)一节中我们介绍了常见的监控数据源。open-falcon作为一个监控框架，可以去采集任何系统的监控指标数据，只要将监控数据组织为open-falcon规范的格式就OK了。

Nginx的数据采集可以通过[ngx_metric](https://github.com/GuyCheung/falcon-ngx_metric)来做。

# 工作原理

ngx_metric是借助lua-nginx-module的`log_by_lua`功能实现nginx请求的实时分析，然后借助`ngx.shared.DICT`存储中间结果。最后通过外部python脚本取出中间结果加以计算、格式化并输出。按falcon格式输出的结果可直接push到falcon agent。

# 使用帮助

详细的使用方法常见：[ngx_metric](https://github.com/GuyCheung/falcon-ngx_metric)
