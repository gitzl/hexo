---
title: Kubernetes-项目中pod调度使用法则
---

##### 前言
kubernetes中部署的pod默认根据资源使用情况自动调度到某个节点。可在实际项目的使用场景中都会有更细粒度的调度需求，比如：某些pod调度到指定主机、某几个相关的服务的pod最好调度到一个节点上、Master节点不允许某些pod调度等。使用kubernetes中的节点调度、节点亲和性、pod亲和性、污点与容忍可以灵活的完成pod调度，从而满足项目中较为复杂的调度需求。下面用一些项目中的使用场景聊聊pod调度使用法则。

##### pod调度主要包含以下内容

```
- nodeSelector
- nodeAffinity
- podAffinity
- Taints & tolerations(污点与容忍)
```

<!-- more -->

##### 指定节点调度

```
kubectl get nodes #获取当前Kubernetes集群全部节点

NAME      STATUS    ROLES     AGE       VERSION
dev-10    Ready     <none>    45d       v1.8.6
dev-7     Ready     master    45d       v1.8.6
dev-8     Ready     <none>    45d       v1.8.6
dev-9     Ready     <none>    45d       v1.8.6

```

###### nodeName
通过主机名指定pod调度到指定节点

```
 spec:
      nodeName: dev-8 #设置要调度节点的名称
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: my-nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30

```
###### nodeSelector
kubernetes中常用label来管理集群的资源，nodeSelector可通过标签实现pod调度到指定节点上。

举列：使用nodeSelector将pod调度到dev-9节点上

step1：给dev-9打标签
```
kubectl label nodes dev-9 test=nginx #设置标签
kubectl get nodes --show-labels |grep test=nginx #查询标签是否设置成功
```
setp2：nodeSelector设置对应标签

```
 spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: my-nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeSelector:  #设置标签
        test: nginx
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      
#查询pod已经成功调度到dev-9节点      
kcc get pods -o wide -n ns-team-1-env-1|grep my-nginx
my-nginx-556dcc8c5c-27f7k          1/1       Running                 0          2m        10.244.2.238   dev-9
```
##### nodeAffinity
node 节点 Affinity ，从字面上很容易理解nodeAffinity就是节点亲和性，Anti-Affinity也就是反亲和性。节点亲和性就是控制pod是否调度到指定节点，相对nodeSelector来说更为灵活，可以实现一些简单的逻辑组合。

###### nodeAffinity策略

```
preferredDuringSchedulingIgnoredDuringExecution #软策略，尽量满足
requiredDuringSchedulingIgnoredDuringExecution  #硬策略，必须满足

根据具体一些常用场景感受下

场景1：必须部署到有 test=nginx 标签的节点
spec:
  containers:
  - name: with-node-affinity
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: test
            operator: In
            values:
            - nginx

场景2：最好部署到有test=nginx 标签的节点
spec:
  containers:
  - name: with-node-affinity
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: test
            operator: In
            values:
            - nginx
场景3：不能部署在dev-7,dev-8节点；最好部署到有test=nginx标签的节点
spec:
  containers:
  - name: with-node-affinity
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: NotIn
            values:
            - dev-7
            - dev-8
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: test
            operator: In
            values:
            - nginx
#通过以上场景可以看出使用节点亲和性可灵活的控制pod调度到节点，实现一些有逻辑组合的调度

Kubernetes中的operator提供了下面几种过滤条件:
- In：label 的值在某个列表中
- NotIn：label 的值不在某个列表中
- Gt：label 的值大于某个值
- Lt：label 的值小于某个值
- Exists：某个 label 存在
- DoesNotExist：某个 label 不存
```
##### podAffinity

nodeSelector和nodeAffinity 都是控制pod调度到节点的操作，在实际项目部署场景中，希望根据服务与服务之间的关系进行调度，也就是根据pod之间的关系进行调度，Kubernetes的podAffinity就可以实现这样的场景，podAffinity的调度策略和nodeAffinity类似也有：

requiredDuringSchedulingIgnoredDuringExecution

preferredDuringSchedulingIgnoredDuringExecution

```
场景：希望my-nginx服务my-busybox服务 最好部署在同一个节点上

# my-nginx基本信息
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: my-nginx  #标签 app = my-nginx
  name: my-nginx
  namespace: ns-team-1-env-1
...
...
  
# my-busybox基本信息  
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: my-busybox #标签 app = my-busybox
  name: my-busybox
  namespace: ns-team-1-env-1
...
...

# 在my-busybox设置pod亲和性

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-busybox
  labels:
    app: my-busybox
spec:
  containers:
  - name: my-busybox
    image: nginx
  affinity:
    podAffinity:    #my-busybox 添加pod亲和性，与打了标签app = my-nginx的pod靠近
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - my-nginx
        topologyKey: kubernetes.io/hostname
# 以上例子表示，有一个pod在运行，并且这个pod有标签是app=my-nginx，my-busybox这个pod就会与其调度到同一个节点上，可预见因为上面我们的my-nginx已经调度到dev-9，因此my-nginx和my-busybox都会在dev-9节点上查询到
```
###### Taints &  tolerations
Taints：污点，tolerations：容忍。在实际项目实践中有时候不希望某些服务调度到指定节点上，比如master节点只运行Kubernetes 组件，应用服务不希望调度到master上。这种场景只需要在master节点设置一个污点。


```
#设置污点
kubectl taint nodes dev-7 system=service:NoSchedule
```
设置了污点就不能调度了么？No 如果设置了污点还是希望某些pod能够调度上去，可以给pod针对污点加容忍。
```
#添加容忍
tolerations:
- key: "system"
operator: "Equal"
value: "service"
effect: "NoSchedule"

#添加了容忍system=service , 这样就可以调度到节点设置了污点：system=service:NoSchedule的主机上，其他没有添加容忍的服务就不会调度到打了污点的节点上。

effect 有三种设置
- NoSchedule：pod不能调度到标记了taints的节点
- PreferNoSchedule：pod最好不要调度到标记了taints的节点
- NoExecute：设置了污点，马上踢出该节点中没有设置对应污点的Tolerate设置的pod
```

###### 参考文档
- https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
- https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/




