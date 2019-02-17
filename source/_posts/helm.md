---
title: Kunbernetes利器-helm
---

###### helm是什么？

引用helm官方的英文介绍:
- The package manager for Kubernetes
- Helm is the best way to find, share, and use software built for Kubernetes.

意思就是helm是kubernetes生态中的一个包管理工具，可以快速发现、共享以及为Kubernetes构建应用.
###### 为什么要拥抱helm?

在没使用helm之前，向kubernetes部署应用，我们要依次部署deployment、svc等，步骤较繁琐。况且随着很多项目微服务化，复杂的应用在容器中部署以及管理显得较为复杂，helm通过打包的方式，支持发布的版本管理和控制，很大程度上简化了Kubernetes应用的部署和管理.

###### 本文包含的主要内容

```
- 安装helm
- helm常用命令
- 自定义Chart
- Chart模板
- 参考文档

```

<!-- more -->

###### 安装helm

> helm客户端

下载helm：https://github.com/helm/helm/releases，其中有Mac、Linux、Windows版本，本文以Linux为例.

```
tar -zxvf  helm-v2.9.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
helm help #确认是否安装成功
```
> helm服务端

直接安装:

```
helm init --upgrade
#docker pull gcr.io/kubernetes-helm/tiller

```
tips：安装服务端会拉取：gcr.io/kubernetes-helm/tiller ，遗憾的是国内无法直接访问"gcr.io"等.

使用阿里云镜像：

```
docker pull registry.cn-hangzhou.aliyuncs.com/kube_containers/tiller
docker tag registry.cn-hangzhou.aliyuncs.com/kube_containers/tiller  gcr.io/kubernetes-helm/tiller:v2.9.1
helm init
```
> 确认服务端（tiller）

```
kubectl get pods -o wide -n kube-system |grep tiller
#Running tiller安装成功
```

> 确认客户端和服务端连接成功

```
helm version

Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
#显示client和Server表示客户端和服务连接成功

```
######  helm search

通过helm  search 可查找helm仓库中可用chart

```
helm search mysql  #查找mysql

NAME                              CHART VERSION APP VERSION DESCRIPTION                                       
stable/mysql                      0.8.2         5.7.14      Fast, reliable, scalable, and easy to use open-...
stable/prometheus-mysql-exporter  0.1.0         v0.10.0     A Helm chart for prometheus mysql exporter with...
stable/percona                    0.3.2         5.7.17      free, fully compatible, enhanced, open source d...
stable/percona-xtradb-cluster     0.1.5         5.7.19      free, fully compatible, enhanced, open source d...
stable/phpmyadmin                 0.1.7         4.8.2       phpMyAdmin is an mysql administration frontend    
stable/gcloud-sqlproxy            0.3.6         1.11        Google Cloud SQL Proxy                            
stable/mariadb                    4.2.7         10.1.34     Fast, reliable, scalable, and easy to use open-...

helm install stable/mysql  #安装mysql

```
###### helm的一些常用命令如下

```
helm search 查找可用的Charts
helm inspect 查看指定Chart的基本信息
helm install 根据指定的Chart 部署一个Release到K8s
helm create 创建自己的Chart
helm package 打包Chart，一般是一个压缩包文件


release:
helm list 列出已经部署的Release
helm delete [RELEASE] 删除一个Release. 并没有物理删除， 出于审计需要，历史可查。
helm status [RELEASE] 查看指定的Release信息，即使使用helm delete命令删除的Release.
helm upgrade 升级某个Release
helm rollback [RELEASE] [REVISION] 回滚Release到指定发布序列
helm get values [RELEASE] 查看Release的配置文件值
repo:
helm repo list
helm repo add [RepoName] [RepoUrl]
helm repo update

```

###### 自定义Chart

以jptStore为例自定义Chart

>Chart基本结构

```
helm-Chart-demo-jptstore
 -templates
    -deployment.yaml
    -service.yaml
 -Chart.yaml
 -values.yaml
 -REAME.md
```

Chart.yaml

```
name: hello-Chart
version: 1.0.0

````
deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jpt-test
  namespace: ns-team-1-env-2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: jpt-test
    spec:
      containers:
        - name: jpt-test
          image: registry.cn-hangzhou.aliyuncs.com/mckj/jptstrore
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
              hostPort: 8083
              protocol: TCP

```
镜像：registry.cn-hangzhou.aliyuncs.com/mckj/jptstrore，部署在ns-team-1-env-2命名空间下，映射主机端口为8083以便于部署后访问测试。名为jtp-test的deployment.
service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: jpt-test
  namespace: ns-team-1-env-2
spec:
  ports:
  - name: http-p-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: jpt-test
```
在ns-team-1-env-2下部署一个名为jpt-test的service

>install Chart
```
cd helm-Chart-demo-jptstore #切到templates目录级
helm install .

#确认是否部署成功
kubectl get deploy,svc,pod -n ns-team-1-env-2 |grep jpt-test
```

###### Helm Chart模板
基于上面的Chart我们将镜像等参数提取到values.yaml文件中，在deployment.yaml通过模板访问
>配置文件  values.yaml

```
image:
  repository: registry.cn-hangzhou.aliyuncs.com/mckj/jptstrore
  tag: latest
  pullPolicy: Always
#注意 key和value之间有给空格，不然后面渲染模板语法通不过
```
>deployment.yaml中引用values.yaml中的数据

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: {{.Chart.Name}}   #访问Chart.yaml中的数据
  namespace: ns-team-1-env-2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: jpt-test
    spec:
      containers:
        - name: jpt-test
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 8080
              hostPort: 8083
              protocol: TCP

```

> 执行带有模板的Chart

```
cd helm
helm install .

#模板语法通过会出现下面的输出数据
RESOURCES:
==> v1/Service
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
jpt-test  ClusterIP  10.111.17.249  <none>       8080/TCP  0s

==> v1beta1/Deployment
NAME      DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
jpt-test  1        1        1           0          0s

==> v1/Pod(related)
NAME                       READY  STATUS       RESTARTS  AGE
jpt-test-5bd49d546b-45dz8  0/1    Terminating  0         8d
jpt-test-5bd49d546b-75764  0/1    Pending      0         0s

```
下载本文例子访问 : https://github.com/gitzl/helm-Chart-demo-jptstore

###### 参考文档
* 官方文档  https://docs.helm.sh/
* 中文文档  https://whmzsu.github.io/helm-doc-zh-cn/
* 参考Demo  https://github.com/gitzl/helm/tree/master/docs/examples/nginx

