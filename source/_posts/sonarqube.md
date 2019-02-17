---
title: sonarQube+jenkins持续审查
---

### SonarQube是管理代码质量一个开放平台，可以快速的定位代码中潜在的或者明显的错误，为提高代码质量提供帮助

> 基于Docker部署sonarQube

``` bash
//安装psql 创建数据库：sonar
docker run --name postgresqldb -e POSTGRES_USER=root -e POSTGRES_PASSWORD=root -d postgres
docker exec -it $containerid /bin/sh
psql -Uroot -droot
\l
create database sonar;

//安装sonarqube 
docker run -d --name sonarqube \
    -p 8001:9000 -p 8093:9092 \
    -e SONARQUBE_JDBC_USERNAME=root \
    -e SONARQUBE_JDBC_PASSWORD=root \
    -e SONARQUBE_JDBC_URL=jdbc:postgresql://172.17.0.4:5432/sonar \
    sonarqube

* IP:8001 访问 默认sonarqube用户和密码：admin

``` 
> 汉化sonarqube

* 下载sonar-l10n-zh-plugin-1.20.jar 
* 放在 extensions/plugins , 重启sonarqube

> jenkins集成sonarQube

1、 安装sonarQubeScaner   系统管理->管理插件->可选插件

![](./img/sonar/1.png)
2、 获取sonarQube的token sonarQube->配置->用户->令牌
![](./img/sonar/3.png)
3、 配置sonarQubeServer   系统管理->系统设置

![](./img/sonar/2.png)
4、 配置sonarQube scaner  系统管理->全局工具设置

![](./img/sonar/4.png)
5、 在构建中添加sonarqube，其中src表示源码的目录，build编译代码后的目录

![](./img/sonar/6.png)
6、成功后，可在sonarqube查看扫描代码的结果

![](./img/sonar/7.png)

tips：通过分析结果中的bugs、漏洞、坏味道可提高代码规范和质量.










