---
title: 高并发下-Zuul参数调优
---

##### What is Zuul?

官方介绍:

```
Zuul is the front door for all requests from devices and web sites to the backend of the Netflix streaming application.
As an edge service application, Zuul is built to enable dynamic routing, monitoring, resiliency and security. 
It also has the ability to route requests to multiple Amazon Auto Scaling Groups as appropriate.

#Zuul相当于是设备和网站到Netflix流应用程序后端的所有请求的前门
#Zuul旨在实现动态路由，监控，弹性和安全性等
在微服务架构中常基于Zuul实现服务网关(API Gateway)服务器
```

在项目实践中，使用jemeter多线程并发访问微服务中的接口时候，在Zuul层出现异常、超时等，从而导致整个请求失败。经过实践，通过调整Zuul的参数、设计高可用架构等可提升TPS、QPS。


##### Zuul参数剖析
> routes

zuul下的routes节点可配置路由转发规则
```
zuul：
   routes：
     echo：
       path：/ myusers / **
       serviceId：myusers-service
       stripPrefix：true

myusers-service：
   ribbon：#负载
     NIWSServerListClassName：com.netflix.loadbalancer.ConfigurationBasedServerList
     listOfServers：http：//example1.com,http：//example2.com
     ConnectTimeout：1000 
    ReadTimeout：3000 
    MaxTotalHttpConnections：500 
    MaxConnectionsPerHost：100
```
如上面的配置，HTTP请求中满足 /myusers/** 规则转发到myuser-service服务。结合ribbon，可支持myusers-service多实例的动态负载。实际项目中可集成eureka或者consul等自动获取listOfServers(多实例服务hosts列表)。

> semaphore

在spring cloud Zuul中有2种对路由的隔离机制，其默认的是信号量（semaphore）对路由做隔离，默认值是100，当一个路由请求的信号量高于100就返回500。

```
zuul:
  semaphore:
    max-semaphores: 5000 #设置全部路由最大信号量
  routes:
    orchestration:
      service-id: orchestration
    resource-manager:
      service-id: resource-manager
      semaphore:
        max-semaphores: 5000 #针对单个服务的路由设置最大信号量
```
设置信号量，可在Zuul节点下对所有路由统一设置信号量（semaphore）大小，在实际项目中推荐为每个服务设置不同的信号量（semaphore）。

> ribbon

SpringCloud中ribbon提供负载均衡能力，实际项目中后端不同服务都是多实例，因此从Zuul路由到某个服务也需要支持负载均衡。

```
zuul:
ribbon:
  OkToRetryOnAllOperations:true     #全部请求开启重试机制
  ReadTimeout: 6000                 #请求处理超时时间
  ConnectTimeout: 6000              #请求连接超时时间
  MaxTotalHttpConnections: 1000     #最大http连接数
  MaxConnectionsPerHost: 100        #每个host最大连接数
  MaxAutoRetries: 10                #最大重试次数
  MaxAutoRetriesNextServer: 10      #切换实例的重试次数
  eureka:
    enabled: true
```
在高并发或者后端服务由于网络等原因，导致请求某一瞬间发生故障，也许后端服务只是暂时不可达或者响应比较慢。通过调整响应时间以及重试次数提高请求成功率。

> hystrix

hystrix(熔断)，当通过服务网关（基于Zuul实现）调用后端服务时候，难免会出现网络、响应超时等情况。通过hystrix可断掉与后端服务的连接，防止拖垮网关服务器。也可以通过hystrix实现服务降级，当发生异常时候，通过fallback处理熔断（比如：返回一些用户能看懂的错误提示等）。

```
#关于hystrix的参数很多，这里列举一些常用的参数
#更多参数，阅读：https://github.com/Netflix/Hystrix/wiki/Configuration

hystrix:
  threadpool:
    default:
      coreSize: 1000   #线程池数量
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 60000  #发生熔断的超时时间
          strategy: SEMAPHORE   #隔离策略
          semaphore:
            max-semaphores: 2000 #信号量大小
            
```
##### 高并发下常见Zuul异常
在高并发下，针对不同的系统架构、业务场景。需要自己调整Zuul各组件参数来满足性能需求。我们在使用jemeter进行并发测试，发现Zuul（服务网关）层出现了一些异常信息，解决了这些异常信息，QPS,TPS都提高了不少。

> 无法获取信号量(semaphore异常)

异常信息-1:

```
spring cloud zuul : could not acquire a semaphore for execution and no fallback available.
#无法获取信号量，系统默认每个路由的信号量为100，当后端一个实例且并发大于100就会经常出现这个异常信息
```

调优配置-1:
```
zuul:
  semaphore:
    max-semaphores: 5000
#可根据系统需要支持的并发数适当增加信号量的大小
```
> 超时

异常信息-2:
```
connect time out...
#当并发访问时，有些服务所在主机响应可能会比较慢，或者某些业务本身比较耗时（比如上传一个大文件的接口）。如果在Zuul层设置的超时时间小于足业务的耗时，会导致正常的业务请求失败。
```
调优配置-2：
```
ribbon:
  ReadTimeout: 6000                 #请求处理超时时间
  ConnectTimeout: 6000              #请求连接时间
   
# 根据业务可适当调大超时时间
```

> 熔断

异常信息-3：

```
short-circuited and no fallback available
#并发访问时，后端某些服务发生熔断
```
调优配置-3：
```
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 60000  #发生熔断的超时时间

# 调整熔断超时时间，熔断时间太短，些耗时的业务部不能work
# 熔断时太长，Zuul服务器可能会被拖垮。所以根据具体业务找到一个合适值。

ribbon:
  OkToRetryOnAllOperations:true     #全部请求开启重试机制
  ReadTimeout: 6000                 #请求处理超时时间
  ConnectTimeout: 6000              #请求连接超时时间
  MaxAutoRetries: 10                #最大重试次数
#调整重试次数，实际项目中由于网络或者资源不够，偶尔会出现后端服务不能访问，一次访问失败不能代表后端服务就挂了。
#因此开启重试机制，调整重试次数。在一定时间内，重试几次都失败，我们才认为后端服务挂了。
```
##### Zuul高可用
我们知道无论在Linux或者windows下，一个进程支持的并发数是有限制的，在实际项目中，服务网关作为微服务架构的入口至关重要。Zuul结合服务发现、负载均衡等启动多个实例做到高可用。如下图:

![](./img/zuul/z1.png)

如上图，启动多个Zuul实例，在Zuul前加一层负载（常用nginx做负载），每个实例承担一些请求，整体支持高并发能力也会提高很多。

##### Zuul版本 - 1.x vs 2.x
截止目前Zuul发布了2个版本，在架构和实现方式以及支持并发的情况都有所不同。

> Zuul-1.x

![](./img/zuul/z2.png)
Filter是Zuul的核心，用来实现对外服务的控制。Filter的生命周期有4个，分别是“PRE”、“ROUTING”、“POST”、“ERROR“

- PRE：过滤器在请求经过路由前被调用。利用这种过滤器可实现身份验证。
- ROUTING：过滤器将请求路由到微服务。这种过滤器用于构建发送给后端微服务的请求，使用Apache HttpClient或Netfilx Ribbon请求微服务。
- POST：过滤器在路由到微服务以后执行。这种过滤器可为响应添加标准的HTTP Header、收集统计信息和指标、将响应发送给客户端。
- ERROR：在其他阶段发生错误时执行该过滤器

> Zuul-2.x

![](./img/zuul/z3.png)
2.x的Zuul基于netty处理请求，netty（高性能、异步事件驱动的NIO框架，提供了对TCP、UDP和文件传输的支持，Netty的所有IO操作都是异步非阻塞的）。相对于1.x版本，很明显把同步改成了异步，把阻塞改成了非阻塞。据官方进行的并发测试，2.x相比1.x性能还是提升了不少。不过2.x相对1.x要复杂一些，如果需要支持很高的QPS、TPS，可以尝试下2.x。

总结：
- 1.x 同步阻塞，编程模型简单，社区成熟，通过调整参数能满足生产性能需求
- 2.x 异步非阻塞，相对编程模型复杂，刚出来也许还有些坑(bug)，追求更好性能可以尝试

##### 最后

当高并发情况下，服务网关服务器(Zuul)可通过以下方法提高支持并发的能力。
- 调整Zuul组件参数
- 支持Zuul高可用，多实例
- 选择异步、非阻塞版本

##### 参考文档

Zuul-github ：https://github.com/Netflix/zuul
Zuul-wiki：https://github.com/Netflix/zuul/wiki
blog：https://www.cnblogs.com/lonelyJay/p/10076441.html
blog: http://blog.didispace.com/api-gateway-Zuul-1-zuul-2-how-to-choose
