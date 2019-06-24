---
title: docker的架构和底层技术
date: 2019-03-11 01:19:56
categories: DevOps
tags:
    - docker
---

## 简介
1. 将物理设备和app用docker engine隔离
2. 后台进程dockerd+rest api server+cli接口（docker）（cs架构）
3.docker version
    ```
     client:
     Version:           18.09.6
     API version:       1.39
     Go version:        go1.10.8
     Git commit:        481bc77
     Built:             Sat May  4 02:35:27 2019
     OS/Arch:           linux/amd64
     Experimental:      false
    
    Server: Docker Engine - Community
     Engine:
      Version:          18.09.6
      API version:      1.39 (minimum version 1.12)
      Go version:       go1.10.8
      Git commit:       481bc77
      Built:            Sat May  4 01:59:36 2019
      OS/Arch:          linux/amd64
      Experimental:     false
    ```
4. containers + images + registry
5. 底层技术支持
<!-- more -->
    - namespace；做隔离pid，net，ipc，mnt，uts
    - control groups：做资源控制，内存 cpu等
    - union file systems：container 和image的分层
6. 实验环境介绍

---
## docker image镜像
1. image概念
    - 文件和meta data的集合（root filesystem）
    - 分层，每层都可以添加改变删除文件，成为一个新的image
    - 不同的image可以共享相同的layer
    - image本身是read only
    - linux内核和发行版和基于一些应用软件都可以看做是docker的分层
    - 
    ```
    root@swoole_dev:/home/tb# docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    hello-world         latest              fce289e99eb9        5 months ago        1.84kB
    root@swoole_dev:/home/tb# docker run centos
    Unable to find image 'centos:latest' locally
    latest: Pulling from library/centos
    8ba884070f61: Pull complete 
    Digest: sha256:ca58fe458b8d94bc6e3072f1cfbd334855858e05e1fd633aa07cf7f82b048e66
    Status: Downloaded newer image for centos:latest
    root@swoole_dev:/home/tb# docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    centos              latest              9f38484d220f        3 months ago        202MB
    hello-world         latest              fce289e99eb9        5 months ago        1.84kB

    ```
    - 为啥centos这么小，因为他是基本地于linux kernel的基础之上
    - image的获取方式
        - dockerfile，build
            - from ubuntu：14.04 基于的base kernel
            - label 说明
            - run 执行的命令
            - expose 暴露的端口
            - entrypoint：程序起点，入口
            - docker build -t tongbo/redis: latest .,.代表当前目录
            - 执行build的每一行的id就是一层封装，层之间可以互用
        - pull from registry(类似github,默认的为dockerhub)
            - docker pull ubuntu:14.04
            - docker push （to server）
            - 
            - 
                ```
                root@swoole_dev:/home/tb# docker pull redis:3.2
                3.2: Pulling from library/redis
                f17d81b4b692: Pull complete 
                b32474098757: Pull complete 
                8980cabe8bc2: Pull complete 
                58af19693e78: Pull complete 
                a977782cf22d: Pull complete 
                9c1e268980b7: Pull complete 
                Digest: sha256:43d2f5e7338ef56b3bda52f1ba7b9b58866c07141e834f64267afb51c89e5086
                Status: Downloaded newer image for redis:3.2
                root@swoole_dev:/home/tb# docker image ls
                REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
                centos              latest              9f38484d220f        3 months ago        202MB
                hello-world         latest              fce289e99eb9        5 months ago        1.84kB
                redis               3.2                 87856cc39862        8 months ago        76MB
                root@swoole_dev:/home/tb# docker search redis
                NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
                redis                            Redis is an open source key-value store that…   7029                [OK]                
                bitnami/redis                    Bitnami Redis Docker Image                      114  
                
            ```
        - dockerhub
            - offical
            - 第三方的，pull的时候需要增加用户名/镜像名字


## 制作base image
1. 比如制作一个u2dev的base
2. 小技巧：如何去掉sudo，sudo groupadd docker sudo gpasswd -a vargant docker service docker restart
3. 以hello-world的image为例
    - tag
    - digest（摘要，消化理解）
    - status
    - more ambitious（有野心的，有兴趣的）
    -
        ```
        # 编辑dockerfile文件
        FROM scratch #base的image，所以从开始不需要
        ADD hello / #把helloadd到image的根目录里
        CMD  ["/hello"] #执行脚本命令
        
        #build，根据dockerfile，一共三步
        @swoole_dev:/home/tb/my_docker_helloworld# docker build -t tongbo/hello_world .
        Sending build context to Docker daemon  12.29kB
        Step 1/3 : FROM scratch
         ---> 
        Step 2/3 : ADD hello /
         ---> b89e60e00ca1
        Step 3/3 : CMD ["/hello"]
         ---> Running in 13d1d20bd719
        Removing intermediate container 13d1d20bd719
         ---> 462eb2d91ad7
        Successfully built 462eb2d91ad7
        Successfully tagged tongbo/hello_world:latest
        root@swoole_dev:/home/tb/my_docker_helloworld# 
        
        #build成功，查看结果
        root@swoole_dev:/home/tb/my_docker_helloworld# docker image ls
        REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
        tongbo/hello_world   latest              462eb2d91ad7        55 seconds ago      8.6kB
        centos               latest              9f38484d220f        3 months ago        202MB
        hello-world          latest              fce289e99eb9        5 months ago        1.84kB
        redis                3.2                 87856cc39862        8 months ago        76MB
        root@swoole_dev:/home/tb/my_docker_helloworld# 
        
        # 查看镜像分层（因from scratch，所以这里是两层）
        root@swoole_dev:/home/tb/my_docker_helloworld# docker history 462eb2d91ad7
        IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
        462eb2d91ad7        3 minutes ago       /bin/sh -c #(nop)  CMD ["/hello"]               0B                  
        b89e60e00ca1        3 minutes ago       /bin/sh -c #(nop) ADD file:ab92082ce376d310a…   8.6kB               
        root@swoole_dev:/home/tb/my_docker_helloworld#
        
        # build 自己的镜像时候必须是gcc -static,否则报文件不存在,==这是为啥内==
        # -static 是让 gcc 进行静态编译，也就是把所有都需要的函数库都集成进编译出来的程序上，这个程序就可以不依赖外部的函数库运行了。
        root@swoole_dev:/home/tb/my_docker_helloworld# docker run tongbo/hello_world
        standard_init_linux.go:207: exec user process caused "no such file or directory
        root@swoole_dev:/home/tb/my_docker_helloworld# gcc -static hello.c -o hello
        root@swoole_dev:/home/tb/my_docker_helloworld# docker build -t tongbo/hello_world .
        Sending build context to Docker daemon  916.5kB
        Step 1/3 : FROM scratch
         ---> 
        Step 2/3 : ADD hello /
         ---> Using cache
         ---> 11b009df24b2
        Step 3/3 : CMD ["/hello"]
         ---> Using cache
         ---> 6c539eb137dd
        Successfully built 6c539eb137dd
        Successfully tagged tongbo/hello_world:latest
        root@swoole_dev:/home/tb/my_docker_helloworld# docker run tongbo/hello_world
        hello,world,docker in c
        ```
## 什么是container
1. container是通过image创建（copy）的
2. container是在image上的基础上增加类一层，叫做container layer，后者是可读写 的，注意image是只读的
3. 理解： 类为image，实例为container
4. image负责app的存储和分发，container负责运行
5. 基于image 创建container
    - docker run image 
    - docker container ls：查看当前正在运行的容器
    - docker container ls -a：查看当前运行和已经运行完成退出的
    - 
      ```
            root@swoole_dev:/home/tb/my_docker_helloworld# docker container ls -a
            CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS                         PORTS               NAMES
            51869bc1fcd5        tongbo/hello_world   "/hello"            7 minutes ago       Exited (0) 7 minutes ago                           happy_bardeen

      ```
      - docker run centos:注意一般run会走latest的版本，如果指定类版本，必须加上，否则会先pull一份过来,下面的执行centos ，==也只是走类bin/bash,why？==
      ```
        root@swoole_dev:/home/tb/my_docker_helloworld# docker run centos
        root@swoole_dev:/home/tb/my_docker_helloworld# docker container ls 
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        root@swoole_dev:/home/tb/my_docker_helloworld# docker container ls -a
        CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS                         PORTS               NAMES
        35f4015c37be        centos               "/bin/bash"         20 seconds ago      Exited (0) 18 seconds ago                          gallant_boyd
        51869bc1fcd5        tongbo/hello_world   "/hello"            11 minutes ago      Exited (0) 11 minutes ago                          happy_bardeen

      ```
      - 交互式运行
        - docker run -it centos
        ```
        # 终端1，ununtu环境
        root@swoole_dev:/etc/docker# docker container ls
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        29fae6a620a9        centos              "/bin/bash"         47 seconds ago      Up 46 seconds                           affectionate_meitner
        root@swoole_dev:/etc/docker# docker container ls
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        29fae6a620a9        centos              "/bin/bash"         51 seconds ago      Up 49 seconds                           affectionate_meitner
        root@swoole_dev:/etc/docker# 
        # run -it  centos 效果，-i为interactive，-t为tty，通过执行 docker run --help查看，完成操作后再容器内退出，退出后容器不会运行
        [root@29fae6a620a9 /]# cat /etc/redhat-release
        CentOS Linux release 7.6.1810 (Core) 
        [root@29fae6a620a9 /]#
        ```
    - docker的management commands和commands
        - Management Commands:
        ```
          builder     Manage builds
          config      Manage Docker configs
          container   Manage containers
          engine      Manage the docker engine
          image       Manage images
          network     Manage networks
          node        Manage Swarm nodes
          plugin      Manage plugins
          secret      Manage Docker secrets
          service     Manage services
          stack       Manage Docker stacks
          swarm       Manage Swarm
          system      Manage Docker
          trust       Manage trust on Docker images
          volume      Manage volumes
        ```
        - 一些简写 命令
            - docker rmi  imageid
            - docker rm containerid
            - docker ps -a 当前的container
            - docker container ls -aq  列出所有的container id
            - docker rm $(docker container ls -aq) rm所有的container
            - 结合xargs grep awk 
            - docker container ls -f "status=exited" -q 删除所有exited的container
           
## 构建自己的docker镜像
1. docker container commit
    - Usage:  docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]] Create a new image from a container's changes
    - 简写为 docker commit
2. docker build
    - build an image from a dockerfile
3. 操作步骤
    1. docker run -it centos
    2. yum install vim
    3. exit
    4. docker container ls -a |grep centos
    5. 4中的centos 安装了vim
    6. `docker container ls -a |grep centos
0f5ccf1365eb        centos               "/bin/bash"         3 minutes ago       Exited (0) About a minute ago                       pedantic_gagarin`
    7. 
        ```
        root@swoole_dev:/home/tb# docker commit pedantic_gagarin yaxiaomu/centos_add_vim:default_yaxiaomu_tag
        sha256:3204e122d66ce500790269c1fed291842b6f18c34286647212d9293c9f56cb45
        root@swoole_dev:/home/tb# docker image ls
        REPOSITORY                TAG                    IMAGE ID            CREATED             SIZE
        yaxiaomu/centos_add_vim   default_yaxiaomu_tag   3204e122d66c        10 seconds ago      361MB
        tongbo/hello_world        latest                 6c539eb137dd        17 hours ago        913kB
        centos                    latest                 9f38484d220f        3 months ago        202MB
        redis                     3.2                    87856cc39862        8 months ago        76MB
                ```
    8. 注意centos和centos_add_vim这两个image会共享很多的layer:`9f38484d220f`
    ```
    root@swoole_dev:/home/tb# docker image ls
    REPOSITORY                TAG                    IMAGE ID            CREATED             SIZE
    yaxiaomu/centos_add_vim   default_yaxiaomu_tag   3204e122d66c        3 minutes ago       361MB
    tongbo/hello_world        latest                 6c539eb137dd        17 hours ago        913kB
    centos                    latest                 9f38484d220f        3 months ago        202MB
    redis                     3.2                    87856cc39862        8 months ago        76MB
    root@swoole_dev:/home/tb# docker history 9f38484d220f
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    9f38484d220f        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
    <missing>           3 months ago        /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
    <missing>           3 months ago        /bin/sh -c #(nop) ADD file:074f2c974463ab38c…   202MB               
    root@swoole_dev:/home/tb# docker history 3204e122d66c
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    3204e122d66c        3 minutes ago       /bin/bash                                       160MB               
    9f38484d220f        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
    <missing>           3 months ago        /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
    <missing>           3 months ago        /bin/sh -c #(nop) ADD file:074f2c974463ab38c…   202MB   
    ```
4. 不提倡以上方式创建，提倡用dockerfile，再build
    ```
    #如何在docker image里yum呢，image不是只读的吗
    # 答：会产生临时的container，然后再写，然后再commit
        root@swoole_dev:/home/tb/docker-centos-vim# docker build -t tongbo/centos_add_vim . 
    Sending build context to Docker daemon  2.048kB
    Step 1/2 : FROM centos
     ---> 9f38484d220f
    Step 2/2 : RUN yum install -y vim
     ---> Running in 67aeb36048ff
    Loaded plugins: fastestmirror, ovl
    ...
        Complete!
    Removing intermediate container 67aeb36048ff
     ---> 907325d6fc6b
    Successfully built 907325d6fc6b
    Successfully tagged tongbo/centos_add_vim:latest

    ```
## dockerfile语法梳理和最佳实践
1. FROM [scratch centos ubuntu:14:04] #制作|使用base image
    - 尽量使用官方image
2. LABEL metadata autohr verison description
3. RUN yum install |apt-get update(注意执行命令都会有新的一层layer，尽量合并成一个语句（&&连接，反斜线\换行），减少层数)
4. workdir /root |demo |pwd(如果没有目录会再当前目录自动创建，注意使用绝对目录)
5. ADD把本地文件条件，添加到image的根目录里去，也可以解压缩
    - ADD test.tar.gz/ # 添加到根目录并解压
6. COPY ，大部分情况使用copy，如果添加远程文件用curl 或者wget
7. ENV mysql_version 5.6 # 设置常量,  保证可维护性
    ```
    ENV MYSQL_VERSION 5.6 
    RUN apt-get instlal -y mysql-server= "${MYSSQL_VERSION}" \
    && rm -rf /var/lib/apt/lists/*
    ```
8. volume 和rescource
9. CMD and entrypoint
10. [docker-library on github](https://github.com/docker-library/docs/tree/master/mysql)， [reference](https://docs.docker.com/reference/)

## run vs cmd vs encrypoint
1. run：执行命令并创建新的image layer
2. cmd：设置容器启动后默认执行的命令和参数
    - 如果docker run指定了其他命令，cmd命令会被忽略
        - docker run -it [image] /bin/bash
    - 如果定义类多个cmd，仅有最后一个被执行

3. entrypoint：设置容器启动时运行的命令
    - 不会被忽略，一定会执行，即使指定了其他命令（区别于cmd）
    - 让容器以应用程序或者服务的形式运行
    - 实践：写一个shell脚本作为entrypoint
        - 
4. 两种格式
    - shell格式 run echo "hello"
    - exec格式 ["/bin/echo",'hello']
    - 如果是exec格式，需要显示指定如下
    ```
    FROM centos
    EVN name Docker
    ENTRYPOIN ["/bin/bash","-c","echo hello $name"]
    ```
## image的分发
1. dockerhub
    - docker login
    - docker image push 
    ```
    root@swoole_dev:/home/tb/my_docker_helloworld# docker push yaxiaomu/hello_world:latest
    The push refers to repository [docker.io/yaxiaomu/hello_world]
    096f9105d9f4: Pushed 
    latest: digest: sha256:dc9c69395640d5fd7cb9e4f8bd2bdbf788b206a59e942a2a40577d9b1c089934 size: 527
    root@swoole_dev:/home/tb/my_docker_helloworld# 
    # 注意image必须是dockerid的用户名，否则会说：
    denied: requested access to the resource is denied
    # 本地删除后再次从docker hub上pull
    root@swoole_dev:/home/tb/my_docker_helloworld# docker run yaxiaomu/hello_world
    hello,world,docker in c
    root@swoole_dev:/home/tb/my_docker_helloworld# docker rm yaxiaomu/hello_world
    Error: No such container: yaxiaomu/hello_world
    root@swoole_dev:/home/tb/my_docker_helloworld# docker rmi yaxiaomu/hello_world
    Untagged: yaxiaomu/hello_world:latest
    Untagged: yaxiaomu/hello_world@sha256:dc9c69395640d5fd7cb9e4f8bd2bdbf788b206a59e942a2a40577d9b1c089934
    root@swoole_dev:/home/tb/my_docker_helloworld# docker run yaxiaomu/hello_world
    Unable to find image 'yaxiaomu/hello_world:latest' locally
    latest: Pulling from yaxiaomu/hello_world
    Digest: sha256:dc9c69395640d5fd7cb9e4f8bd2bdbf788b206a59e942a2a40577d9b1c089934
    Status: Downloaded newer image for yaxiaomu/hello_world:latest
    hello,world,docker in c
    root@swoole_dev:/home/tb/my_docker_helloworld# 

    ```
2. 因为安全因素考虑，分享image不如分享Dockerfile
    - 可以通过和github关联，自动拉取指定项目下的dockerfile，自动build
    - [私有本地仓库搭建，但没有图形化界面：](https://hub.docker.com/_/registry)docker run -d -p 5000:5000 --restart always --name registry registry:2 
        - 可以向指定私有库提交docker built -t serverip:port/name:tag
        - 安全性修改，创建文件/etc/docker/daemon.json deamon.json,配置加入 insecure-registries: ip :端口
        - 再修改 root@swoole_dev:/etc/init.d# vim  /lib/systemd/system/docker.service,增加一行：EnvironmentFile=/etc/docker/daemon.json
        - 重启docker服务 service docker restart
        - 通过docker registry api 查看 ，http查看
        - 记录在了segmentfault
        

---

## dockerfile实战

---

1. flask demo，把python程序打包成image，运行container
    - 准备一个带pyhton的base image
    - 需要安装flask
    - 需要运行起来app
2. 操作步骤
    ```
    root@swoole_dev:/home/tb/flask_demo# more app.py 
    ## app.py
    from flask import Flask
    app = Flask(__name__)
    @app.route('/')
    def hello():
        return "hello,tb de docker"
    if __name__ == '__main__':
        app.run()

    # 安装软件
    apt-get install python-minimal
    apt install python-pip
    pip install flask
    
    # 运行结果
       root@swoole_dev:/home/tb/flask_demo# python app.py 
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
       127.0.0.1 - - [22/Jun/2019 21:00:05] "GET / HTTP/1.1" 200 -
    
    #dockerfile ,注意cppy的 app.py为写成类绝对路径报错类，那就转移到当前目录下吧。
    FROM python:2.7
    LABEL maintainer="tongbo<demo.com@126.com>"
    RUN pip install flask
    COPY app.py /app/
    WORKDIR /app
    EXPOSE 5000
    CMD ["pyhton","app.py"]

    # build
    root@swoole_dev:/home/tb/flask_hello_world# docker build -t yaxiaomu/flask_demo:latest .
    Sending build context to Docker daemon  3.072kB
    Step 1/7 : FROM python:2.7
     ---> 37093962fbf5
    Step 2/7 : LABEL maintainer="tongbo<demo.com@126.com>"
     ---> Using cache
     ---> c4ac0caa5aab
    Step 3/7 : RUN pip install flask
     ---> Using cache
     ---> 60c7e35f23a3
    Step 4/7 : COPY app.py /app/
     ---> a7a69c1da0b6
    Step 5/7 : WORKDIR /app
     ---> Running in 2122fe24efd6
    Removing intermediate container 2122fe24efd6
     ---> f6b586c33cbc
    Step 6/7 : EXPOSE 5000
     ---> Running in e368df4c5205
    Removing intermediate container e368df4c5205
     ---> 6a7e1858c5e7
    Step 7/7 : CMD ["pyhton","app.py"]
     ---> Running in e8c96756cc9e
    Removing intermediate container e8c96756cc9e
     ---> c37bb4c557da
    Successfully built c37bb4c557da
    Successfully tagged yaxiaomu/flask_demo:latest
    
    ## 运行container，报错
    root@swoole_dev:/home/tb/flask_hello_world# docker run  yaxiaomu/flask_demo
    docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "exec: \"pyhton\": executable file not found in $PATH": unknown.
    ERRO[0000] error waiting for container: context canceled 
    ```
3.  debug
    - 针对创建临时中间状态的image，根据image id进入/bin/bash
    - docker run it imageid /bin/bash
    - cd .. && begin your debug
    
    ```
    root@swoole_dev:/home/tb/flask_hello_world# docker run -it c37bb4c557da /bin/bash
    root@a972581ff13e:/app#
    root@a972581ff13e:/app# python app.py 
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
     ## 看结果明明可以运行，再看报错，原来是python写成pyhton了。改一下dockerfile，成功了
      
      root@swoole_dev:/home/tb/flask_hello_world# docker run  yaxiaomu/flask_demo 
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
     ## 后台运行 docker run -d,--name 增加名字，删除启动时可用
     root@swoole_dev:/home/tb/flask_hello_world# docker run -d --name tb_demo yaxiaomu/flask_demo
     e7841af659ab469f598151b0f43c1333a77a52c6610d6894a5dfbec887d6848e

    ```
## 容器的操作
1. `docker container stop 664f2033265b | docker stop 664
664f2033265b
[1]+  Exit 137                docker run yaxiaomu/flask_demo`
2. `root@swoole_dev:/home/tb/flask_hello_world# docker exec -it e7841af659ab /bin/bash
root@e7841af659ab:/app# `
3. docker exec -it e7841af659ab ip a
4. docker rm $(docker ps -aq)
5. docker start|stop demo
6. docker inspect containerId # 查看完整追踪
7. docker container logs containerid
8. docker container commands...

## dockerfile实战2
1. stress工具
2. apt-get install stress
3. 测试主机 或者容器资源（内存、cpu等）
4. 每个docker启动的时候都可以限制cpu 内存等
```
FROM UBUNTU
RUN apt-get update && apt-get install -y stress
ENTRYPOINT ["usr/bin/stress"]
CMD 

## 运行
docker run -it yaxiaomu/ubuntu_stress --vm 1 --verbose
```

## 容器的资源限制
1. 物理机-虚拟机之间的资源配置 virtualbox
2. -m 限制memory swap memory
3. -c cpu shares relate weight,相对两倍权重
4. docker run --cpu-shares=5 --name=test2 --cpu1
5. docker run --cpu-shares=10 --name=test3 --cpu1
6. control groups，分层layer通过union file system实现

## 完

    


       