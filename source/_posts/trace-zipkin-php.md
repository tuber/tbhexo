---
title: php的调用链追踪入门（zipkin）
date: 2018-07-23 23:16:44
categories: PHP
tags:
    - traceing
    - zipkin
---
## 学名（Distributed TracingSystem 分布式追踪系统）
## [支持的客户端语言情况](https://zipkin.io/pages/existing_instrumentations.html)
## [奉上谷歌论文Dapper](https://bigbully.github.io/Dapper-translation/)
## [新美大之CAT（开源）](https://tech.meituan.com/CAT_in_Depth_Java_Application_Monitoring.html)
## [大厂子自己造的轮子](https://www.jianshu.com/p/e02972487e00)
## 华为专利 [link](https://note.youdao.com/)
## 安装运行

1. 代码执行
```
root@es_002:/home/tb/tbdown/zipkin# ls
quickstart.sh  zipkin-server-2.11.7-exec.jar
root@es_002:/home/tb/tbdown/zipkin# java -jar zipkin-server-2.11.7-exec.jar 
                                    ********
                                  **        **
                                 *            *
                                **            **
                                **            **
                                 **          **
                                  **        **
                                    ********
                                      ****
                                      ****
        ****                          ****
     ******                           ****                                 ***
  ****************************************************************************
    *******                           ****                                 ***
        ****                          ****
                                       **
                                       **


             *****      **     *****     ** **       **     **   **
               **       **     **  *     ***         **     **** **
              **        **     *****     ****        **     **  ***
             ******     **     **        **  **      **     **   **

:: Powered by Spring Boot ::  

```
 <!-- more -->
3. [Zipkin of php](https://github.com/openzipkin/zipkin-php)类库[了解](https://zipkin.io/pages/existing_instrumentations.html)一下


```
  //如果是新项目需要引入包，按需参考以下命令
  composer init
  composer config -g secure-http false

  //参考官方php类库 composer.json文件，得到以下
  root@udev:/home/tb/tbtmp# ls
  composer.json  composer.lock  vendor
  root@udev:/home/tb/tbtmp# cd vendor/
  root@udev:/home/tb/tbtmp/vendor# ls
  autoload.php  composer  evenement   jcchavezs  phpdocumentor  phpunit  react        sebastian  symfony
  bin           doctrine  guzzlehttp  myclabs    phpspec        psr      ringcentral  squizlabs  webmozart
  root@udev:/home/tb/tbtmp/vendor# 
```


4. 数据持久化到mysql

```
create database zipkin;

CREATE TABLE IF NOT EXISTS zipkin_spans (
  `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` BIGINT NOT NULL,
  `id` BIGINT NOT NULL,
  `name` VARCHAR(255) NOT NULL,
  `parent_id` BIGINT,
  `debug` BIT(1),
  `start_ts` BIGINT COMMENT 'Span.timestamp(): epoch micros used for endTs query and to implement TTL',
  `duration` BIGINT COMMENT 'Span.duration(): micros used for minDuration and maxDuration query'
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_spans ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `id`) COMMENT 'ignore insert on duplicate';
ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`, `id`) COMMENT 'for joining with zipkin_annotations';
ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTracesByIds';
ALTER TABLE zipkin_spans ADD INDEX(`name`) COMMENT 'for getTraces and getSpanNames';
ALTER TABLE zipkin_spans ADD INDEX(`start_ts`) COMMENT 'for getTraces ordering and range';

CREATE TABLE IF NOT EXISTS zipkin_annotations (
  `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.trace_id',
  `span_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.id',
  `a_key` VARCHAR(255) NOT NULL COMMENT 'BinaryAnnotation.key or Annotation.value if type == -1',
  `a_value` BLOB COMMENT 'BinaryAnnotation.value(), which must be smaller than 64KB',
  `a_type` INT NOT NULL COMMENT 'BinaryAnnotation.type() or -1 if Annotation',
  `a_timestamp` BIGINT COMMENT 'Used to implement TTL; Annotation.timestamp or zipkin_spans.timestamp',
  `endpoint_ipv4` INT COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_ipv6` BINARY(16) COMMENT 'Null when Binary/Annotation.endpoint is null, or no IPv6 address',
  `endpoint_port` SMALLINT COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_service_name` VARCHAR(255) COMMENT 'Null when Binary/Annotation.endpoint is null'
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_annotations ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `span_id`, `a_key`, `a_timestamp`) COMMENT 'Ignore insert on duplicate';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`, `span_id`) COMMENT 'for joining with zipkin_spans';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTraces/ByIds';
ALTER TABLE zipkin_annotations ADD INDEX(`endpoint_service_name`) COMMENT 'for getTraces and getServiceNames';
ALTER TABLE zipkin_annotations ADD INDEX(`a_type`) COMMENT 'for getTraces';
ALTER TABLE zipkin_annotations ADD INDEX(`a_key`) COMMENT 'for getTraces';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id`, `span_id`, `a_key`) COMMENT 'for dependencies job';

CREATE TABLE IF NOT EXISTS zipkin_dependencies (
  `day` DATE NOT NULL,
  `parent` VARCHAR(255) NOT NULL,
  `child` VARCHAR(255) NOT NULL,
  `call_count` BIGINT,
  `error_count` BIGINT
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_dependencies ADD UNIQUE KEY(`day`, `parent`, `child`);

```
以mysql为存储方式启动(222为另外一台机器)：

```
STORAGE_TYPE=mysql MYSQL_HOST=192.168.50.222 MYSQL_TCP_PORT=3306 MYSQL_DB=zipkin MYSQL_USER=root MYSQL_PASS='root' java -jar zipkin-server-2.11.7-exec.jar
```



```
Database changed
mysql> show tables;
+---------------------+
| Tables_in_zipkin    |
+---------------------+
| zipkin_annotations  |
| zipkin_dependencies |
| zipkin_spans        |
+---------------------+
3 rows in set (0.00 sec)

mysql> select * from zipkin_spans;
+---------------+------------------+------------------+--------------+------------------+-------+------------------+----------+
| trace_id_high | trace_id         | id               | name         | parent_id        | debug | start_ts         | duration |
+---------------+------------------+------------------+--------------+------------------+-------+------------------+----------+
|             0 | 1540542210608961 | 1540542210608963 | /method_of_a |             NULL |      | 1540542210608963 |   380025 |
|             0 | 1540542210608961 | 1540542210608964 | /method_of_b | 1540542210608963 |      | 1540542210608968 |   229175 |
|             0 | 1540542210608961 | 1540542210838146 | mysql.user   | 1540542210608963 |      | 1540542210838148 |   100442 |
+---------------+------------------+------------------+--------------+------------------+-------+------------------+----------+
3 rows in set (0.00 sec)
```
与下图匹配
![image][1]

    1. 在10ms的时候，client send发起一个请求
    2. 服务端在9ms后(10+9),之后，收到这个请求 server receive
    3. 12ms后，server处理完了业务逻辑，返回给客户端 server send
    4. 1ms后，client收到了这个响应 client receive

## 实战代码

1. new trace是一个span的名称，这三个span是同级别
```
        $span_root = $tracer->newTrace();
        $span_root->setName('pre_con_redis');
        $span_root->start();
        try {
            //1000毫秒 =1秒=1000000微秒 usleep 单位是微妙
           usleep(1000000);
        } finally {
            $span_root->finish();
        }

        //new trace是一个span的名称
        $span_root_2 = $tracer->newTrace();
        $span_root_2->setName('do_redis');
        $span_root_2->start();
        try {
            //1000毫秒 =1秒=1000000微秒 usleep 单位是微妙
            usleep(2000000);
        } finally {
            $span_root_2->finish();
        }

        $span_root_3 = $tracer->newTrace();
        $span_root_3->setName('do_redis');
        $span_root_3->start();
        try {
            //1000毫秒 =1秒=1000000微秒 usleep 单位是微妙
            usleep(3000000);
        } finally {
            $span_root_3->finish();
        }

```
对应下面的1s，2s，3s  

![image][2]

2. 父子关系

```
  $span_root = $tracer->newTrace();
        $span_root->setName('php_demo_begin');
        $span_root->start();
        $span_root->tag('http.status_code', '200');
        try {
            $parentContext = $span_root->getContext();
            $child_span1=$tracer->newChild($parentContext);
            $child_span1->setName("pre_con_redis");
            $child_span1->start();
            //1000毫秒 =1秒=1000000微秒 usleep 单位是微妙
           usleep(1000000);
        } finally {
            $child_span1->finish();
        }

        /**new trace是一个span的名称
        *在每个节点处（span）打点，初始化时设置context上下文（暂时忽略这个概念），name（span名称）
        *这样就产生了一条span记录，包含：context，span-name，start-time，end-time。
         **/
        $child_span1 = $tracer->newChild($parentContext);
        $child_span1->setName('do_redis');
        $child_span1->start();
        $span_root->tag('http.status_code', '200');
        try {
            //1000毫秒 =1秒=1000000微秒 usleep 单位是微妙
            usleep(2300000);
        } finally {
            $child_span1->finish();
        }

        $child_span3 = $tracer->newChild($parentContext);
        $child_span3->setName('redis_return');
        $child_span3->start();
        $span_root->tag('http.status_code', '200');
        try {
            //1000毫秒 =1秒=1000000微秒 usleep 单位是微妙
            usleep(1200000);
        } finally {
            $child_span3->finish();
        }

        $span_root->finish();
```
 
 ## 实际问题
 1. 如果做到对现有代码的低侵入
    对中间件、类库的二次包装？
 2. 如果兼顾代码执行效率、性能、稳定性
    根据load动态调整？
 3. 扩展及降级怎么最方便
 4. 该记录什么信息？怎么记录？hook
    ![image][3]
 5. [和ELK结合](https://dzone.com/articles/distributed-tracing-with-zipkin-and-elk/)
 
 ## [点击查看完整多图](http://note.youdao.com/noteshare?id=16e9b0e6405d9b23ee0d38b52cad6a4f)

[1]: /img/php/zipkin_php_01.jpg
[2]: /img/php/zipkin_php_02.jpg
[3]: /img/php/zipkin_php_03.jpg

