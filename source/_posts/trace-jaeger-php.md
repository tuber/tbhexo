---
title: php的调用链追踪入门（jeager）
date: 2018-08-13 23:26:44
categories: PHP
tags:
    - traceing
    - jaeger
---

## 简单介绍
- 用途：监控monitor 检测troubleshoot  事务transations 处理在复杂的分布式系统中 complex distributed systems 性能优化分析
- [官网地址](https://www.jaegertracing.io/)
- 背景：微服务（microservices）的发展、网络的不稳定、多环的服务请求记录错误排查定位
- 相关：
    - 轻量级标准化层API：[OpenTracing](http://opentracing.io/)
    - [jagger前世今生](https://juejin.im/entry/58c3bf8761ff4b005d90c5e1)
    - [ 阿里好文](https://yq.aliyun.com/articles/514488)
    - [openttacing  api 中文版介绍](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md)
    - [七牛](https://developer.qiniu.com/insight/manual/5054/all-link-track-profile)
    - [阿里好文2](https://yq.aliyun.com/articles/514488?utm_content=m_43347)
    - [google 论文中文](https://bigbully.github.io/Dapper-translation/)
<<<<<<< HEAD
- 两对儿图：
    <!-- more -->
    - 日常调用（逻辑链路）
![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/E9F2F681E5C54C8B8E806DD9A30EEE1A)
![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/2460C8742ACF4881A887A87511D74A12)
    - 链路分析（物理链路）
![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/DDE2BE398CBF45B4A19C95E17A1B2EF7)
![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/8C10C46800F54669B10E7376EAC4E536)

- 整图  
![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/55C38C3F3E0B43ECA926A6FD1CC25E9E)

=======
 <!-- more -->
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
## 名词介绍
1. TrancesID 全局跟踪id，标记一次完整的服务调用（可包含多次子，子子调用） 一般可以设置为TraceId和顶级span相同
2. SpansID 一次方法（程序块、RPC、db）的调用，每一个RPC对应到一个单一的span，但是traceId都是相同的。
    注意：一个节点上可能有多次span（可能此端相对上游来说是服务端，接收和应答上游服务。对于下游（下游）可能有多个，又会是调用和接收下游的服务），不能把一个节点上的多个span合并成一个。
     一个节点上并不总是发生完整的cs sr ss cr这四种事件。因为并不是所有的节点都有上游服务和下游服务。
3. Spans Tags span日志集合（key-value）
4. Baggage Item Trace层面的日志数据
5. Refererneces Span之间的关系
6. grpc protobuf thrift 跨语言通信框架（协议）
7. 事件（annotation数据）类型
    1. cs ：客户端/消费者发起请求 client send
    2. cr ：客户端/消费者收到应答 client recevive
    3. sr ：服务端/生产者接收到请求 server receive
    4. ss ：服务端/生产者发送应答 server send
        - Annotation的意思是注释，备注，很好理解。可以理解为一个Hash结构，长度最多为4.每种关键事件包含value，  timestamp，和endpoint。value为以上四种事件类型之一，timestamp即为发生调用时间。enpoint用于记录发生的机器ip和服务名称servername。所以一般cs和cr的机器名称相等，sr和ss的机器名称相同
8. duration 时间消耗总耗时
9. timestamp 一般用于记录时间消耗的起点
10. endpoint 记录终端机器发生的ip和名称（可以是相对的发起方和调用方），也会把endpoint数据放在annotation里。
11. BinaryAnnotation 除去以上的时间、事件、节点等信息，如果还需要++绑定业务数据++（日志、异常），将数据写入到BinaryAnnotion中。结构和Annotation一样。
12. parentId 父span的Id，当然具有层级关系。顶级span（最先接触服务调用入口）是没有parentId的
13. span name 一般为接口的方法名
<<<<<<< HEAD
14. 一图胜三言(下图)
15. ![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/7B603AE043744222A2FABDBEC691C32E)
=======
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99

## 大概流程
1. 请求入口生成唯一traceId，用于贯穿整个服务，
2. 在调用具体的服务（或方法）前生成span（spanid可以平行、嵌套，），记录调用时间
3. 在调用具体的服务（或方法）时，传参1的traceId和2的spanid，调用http server或者rpc server
4. 调用完成后关联到1的traceId
5. 统一存储（涉及各编程语言调用类库及客户端收集存储）
6. 查询结果，输出展示

<<<<<<< HEAD

=======
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
## 应用场景
- [做一个筛选条件的导出excel操作](https://blog.yeeef.com/post/optimize-slow-request/)，筛选条件经过sql查询生成对象数组，然后经过代码逻辑处理导出csv，查看是否有redis缓存，最后到客户端输出。可以查看是sql问题（数据查询和数据转换）？redis连接缓存问题？客户端导出慢？
- 谷歌首页搜索一个词语，需要毫秒级响应。列表，图片，推荐，所有的不只一个接口。如果时间太长，到底是哪里慢？

## 组成部分
- 前端UI展示
- 数据持久化存储（Cassandra or elastic）
- 数据查询组件
- 客户端库library（Go, Node, Java, Python）
- 客户端代理anent（可控制应用追踪数据采样）
- 数据收集处理collector

## 数据收集展示流程

    ```
    Agent      --> Collector    # 从Agent发送数据到Collector
    Collector  --> Cassandra    # 从Collector写数据到Cassandra
    Query      --> Cassandra    # 从Cassandra读数据到Query
    
    ```

## ALL IN ONE DOCKER IMAGE  
1. 仅供测试环境，数据放在内存中了 -docker images 

    ```
     docker run -d --name jaeger \
          -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
          -p 5775:5775/udp \
          -p 6831:6831/udp \
          -p 6832:6832/udp \
          -p 5778:5778 \
          -p 16686:16686 \
          -p 14268:14268 \
          -p 9411:9411 \
          jaegertracing/all-in-one:latest
    ```
<<<<<<< HEAD
2.  ![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/2EC21DEEF68B4BA5B855FBEA04585B4E)
=======
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
  
## 模拟测试数据方法一： 
 
- [Hot R.O.D. Rides OnDemand](https://github.com/jaegertracing/jaeger/tree/master/examples/hotrod)

    ```
        root@docker_001:/home/tb# docker images
        REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
        jaegertracing/example-hotrod   latest              73985981101a        12 hours ago        15.5MB
        jaegertracing/all-in-one       latest              8ab45e1d2c89        4 days ago          40.4MB
        redislabs/rebloom              latest              3386f3e4e33a        4 weeks ago         83.6MB
        tb_php-fpm                     latest              ddd54dd40a46        4 months ago        225MB
        nginx                          latest              c5c4e8fa2cf7        4 months ago        109MB
        phpdockerio/php56-fpm          latest              e944c32c61aa        5 months ago        225MB
        hello-world                    latest              f2a91732366c        9 months ago        1.85kB
        root@docker_001:/home/tb# docker run   --rm   --link jaeger   -p8080-8083:8080-8083   jaegertracing/example-hotrod:latest   all   --jaeger-agent.host-port=jaeger:6831
        2018-08-28T06:44:27.000Z	INFO	cmd/root.go:86	Using expvar as metrics backend
        2018-08-28T06:44:27.000Z	INFO	cmd/all.go:25	Starting all services
        2018-08-28T06:44:27.104Z	INFO	log/logger.go:37	Starting	{"service": "route", "address": "http://0.0.0.0:8083"}
        2018-08-28T06:44:27.105Z	INFO	log/logger.go:37	Starting	{"service": "frontend", "address": "http://0.0.0.0:8080"}
    
    ```
<<<<<<< HEAD
- ![image](https://note.youdao.com/yws/public/resource/40533ca20a9edc5b60490f3bd03e3562/1E1DCDC593AC48B4A7FE7283E55601E7?ynotemdtimestamp=1561304251618)

## 模拟数据方法二 go源码安装
-    ```
=======

## 模拟数据方法二 go源码安装

    ```
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
        go get github.com/jaegertracing/jaeger
        cd $GOPATH/src/github.com/jaegertracing/jaeger
        make install 
    
    ```
<<<<<<< HEAD
    
- 问题：glide (go 包管理工具)的安装
    
=======
   
## 问题：glide (go 包管理工具)的安装

>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
    ```
     /bin/sh: 1: glide: not found
        glide install
        make: glide: Command not found
        Makefile:158: recipe for target 'install' failed
        make: *** [install] Error 127
        
        ## 安装glide
        curl https://glide.sh/get | sh
          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                         Dload  Upload   Total   Spent    Left  Speed
        100  4833  100  4833    0     0    944      0  0:00:05  0:00:05 --:--:--  1192
        ARCH=amd64
        OS=linux
        Using curl as download tool
        Getting https://glide.sh/version
        TAG=v0.13.1
        GLIDE_DIST=glide-v0.13.1-linux-amd64.tar.gz
        Downloading https://github.com/Masterminds/glide/releases/download/v0.13.1/glide-v0.13.1-linux-amd64.tar.gz
        glide not found. Did you add $GOBIN to your $PATH?
        Fail to install glide
        root@docker_001:/home/tb/go_work/src/github.com/jaegertracing/jaeger#
<<<<<<< HEAD
    
    ``` 
    
解决问题： 

    `vim .profile` # 增加GOBIN的PATH  

然后 

    ```
=======
        vim .profile # 增加GOBIN的PATH  
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
        source .profile 
        root@docker_001:/home/tb# echo $GOBIN
        /usr/local/go/bin
        root@docker_001:/home/tb# cd /home/tb/go_work/src/github.com/jaegertracing/jaeger
        root@docker_001:/home/tb/go_work/src/github.com/jaegertracing/jaeger# 
        root@docker_001:/home/tb/go_work/src/github.com/jaegertracing/jaeger# curl https://glide.sh/get | sh
          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                         Dload  Upload   Total   Spent    Left  Speed
        100  4833  100  4833    0     0    481      0  0:00:10  0:00:10 --:--:--  1120
        ARCH=amd64
        OS=linux
        Using curl as download tool
        Getting https://glide.sh/version
        TAG=v0.13.1
        GLIDE_DIST=glide-v0.13.1-linux-amd64.tar.gz
        Downloading https://github.com/Masterminds/glide/releases/download/v0.13.1/glide-v0.13.1-linux-amd64.tar.gz
        glide version v0.13.1 installed successfully

<<<<<<< HEAD
然后执行
=======
        #然后执行
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99

       `make install`
       `cd examples/hotrod`
       `go run ./main.go all`,执行结果如下  
   
       [INFO]	--> Exporting go.uber.org/multierr
       [INFO]	--> Exporting go.uber.org/zap
       [INFO]	Replacing existing vendor dependencies
       root@docker_001:/home/tb/go_work/src/github.com/jaegertracing/jaeger# cd examples/hotrod
       root@docker_001:/home/tb/go_work/src/github.com/jaegertracing/jaeger/examples/hotrod# go run ./main.go all
       2018-08-30T11:57:49.524+0800	INFO	cmd/root.go:86	Using expvar as metrics backend
       2018-08-30T11:57:49.524+0800	INFO	cmd/all.go:25	Starting all services
       2018-08-30T11:57:49.625+0800	INFO	log/logger.go:37	Starting	{"service": "route", "address": "http://0.0.0.0:8083"}
       2018-08-30T11:57:49.626+0800	INFO	log/logger.go:37	Starting	{"service": "frontend", "address": "http://0.0.0.0:8080"}
       2018-08-30T11:57:49.728+0800	INFO	log/logger.go:37	Starting	{"service": "customer", "address": "http://0.0.0.0:8081"}
       2018-08-30T11:57:49.729+0800	INFO	log/logger.go:37	TChannel listening	{"service": "driver", "hostPort": "[::]:8082"}
       ...
<<<<<<< HEAD
    
   ```
也是同样通过8080端口访问即可。

=======
    ```
也是同样通过8080端口访问即可。
>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99
## todo
1. elasticsearch 存储信息
2. php 调用追踪
3. [源码安装jaeger](https://imscc.io/posts/trace/install_jaeger_on_linux/)
4. zipkin

<<<<<<< HEAD
=======
## [高清大图](http://note.youdao.com/noteshare?id=40533ca20a9edc5b60490f3bd03e3562)

>>>>>>> 5cf5fc3ccec8342646523cf4982e8108d5a47e99

