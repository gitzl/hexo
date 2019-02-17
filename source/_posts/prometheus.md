---
title: prometheus+grafana+alertmanger - 监控方案
---

##### 监控方案涉及的关键问题

* 采集数据
* 存储数据
* 告警、展示数据

那么基于prometheus的监控方案是用什么技术实现？本文基于docker进行部署，目的以简单的demo快速了解prometheus监控方案的涉及的知识点以及流程。

##### 本文介绍的内容
* prometheus整体架构介绍
* 安装prometheus
* 安装grafana
* 安装alertmanager
* grafana关联prometheus
* prometheus关联alertmanager
* prometheus定义告警规则
* alertmanager触发告警



##### prometheus整体架构
![](./img/prom/p1.png)
从prometheus官方给的架构图分析出，基于prometheus的监控方案常用组件：
* Exporters ：暴露metrics,收集监控指标，并以一种规定的数据格式提供给Prometheus-采集监控对象数据
* Prometheus Server ：收集数据和存储数据到时间序列数据库中，收集的数据由Exporters提供-采集/存储数据
* Alertmanager ：告警管理，接收Prometheus的告警，去重/分组/发出告警（邮件、webhook等）- 告警
* Grafana：监控Dashbord，UI展示，设置Prometheus Server地址即可自定义监控Dashbord- UI展示
* Push Gateway： 用于短期的jobs，jobs直接向Prometheus server端推送它们的 metrics.用于服务层面的 metrics

##### 部署组件
本文以docker方式部署：prometheus、grafana、alertmanager

###### install prometheus
````
docker run --name prometheus -d -p 9090:9090  quay.io/prometheus/prometheus
# 暴露端口：9090
docker run --name prometheus -d -p 9090:9090 -v  /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml quay.io/prometheus/prometheus
# 挂载prometheus.yml文件，便于在主机上直接修改
docker run --name prometheus -d -p 9090:9090 --link=alertmanger:alertmanger -v  /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /etc/prometheus/rules:/etc/prometheus/rules  quay.io/prometheus/prometheus
# 挂载挂载prometheus.yml文件，创建了rules目录，rules存放告警规则yaml文件，后续会提到
````

###### install grafana

````
docker run -d -p 3000:3000 --name=grafana grafana/grafana

````

###### install alertmanger

````
docker run --name alertmanger -d  -p 9093:9093  -v /etc/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml quay.io/prometheus/alertmanager
#挂载告警文件：alertmanager.yml，便于在主机上直接修改文件

````
##### grafana关联prometheus

![](./img/prom/p2.png)
Prometheus界面，采集promhttp_metric_handler_requests_total为demo进行演示，这个metrics是prometheus自己监控自己的http请求数量
![](./img/prom/p3.png)
* 访问grafana，默认用户名和密码都是：admin
* 设置-type选择Prometheus-填写Prometheus访问地址，点击Save&Test 测试关联prometheus是否成功
![](./img/prom/p4.png)
* 新建dashboard，设置Metrics，即可展示数据

##### prometheus关联alertmanager
prometheus配置文件路径：/etc/prometheus/prometheus.yaml，在如下配置设置告警地址

````
global:
  scrape_interval:     15s 
  evaluation_interval: 15s 
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanger:9093']
# 关联告警服务器alertmanger    
scrape_configs:
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['localhost:9090']
# prometheus 服务监听的端口

````

  查看prometheus-Status-configuration，检测配置是否生效
![](./img/prom/p5.png)

##### prometheus定义告警规则-prom_rules.yml

```
groups:
- name: test-rule
  rules:
  - alert: promReqCounts
    expr: promhttp_metric_handler_requests_total > 10
    for: 0s
    labels:
      prom: http
    annotations:
      summary: High prometheus request total is above 1000
```
* 定义request请求总数大于10就发生告警，标签为：prom：http

* 定义rules考官网例子：https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/

在prometheus中关联rule文件 - prom_rules.yml

```
global:
  scrape_interval:     15s 
  evaluation_interval: 15s 

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanger:9093']
# 关联告警服务器alertmanger    
rule_files:
    - "rules/prom_rules.yml"
# 定义告警规则：prom_rules.yml
scrape_configs:
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['localhost:9090']
# prometheus 服务监听的端口

```

##### alertmanager触发告警

配置alertmanager.yml文件

````
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.test'
  routes:
  - receiver: 'web.hook'
    match:
      prom: http
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:8888'
- name: 'web.test'
  webhook_configs:
  - url: 'http://127.0.0.1:88888'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
````
* 当alertmanger接收到标签为：prom:http的告警就触发 web.hook，当然也可以发邮件等操作
* 更多告警参考官网例子：https://prometheus.io/docs/alerting/configuration/

如果发生了告警，alertmaneger界面会有记录

![](./img/prom/p6.png)


##### 参考文档
- prometheus : https://prometheus.io/docs/prometheus/latest/installation/
- exporter : https://prometheus.io/docs/instrumenting/exporters/#exporters-and-integrations
- alertmanger: https://prometheus.io/docs/alerting/alertmanager/
- prometheus rules : https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/
- grafana：http://docs.grafana.org/