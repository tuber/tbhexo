---
title: docker环境的各种搭建方法
date: 2018-07-25 23:17:33
categories: DevOps
tags:
    - docker
---
1. install docker
    - 登录
    - id.docker.com
    - download win desktop:[必须win10](https://docs.docker.com/docker-for-windows/install/)& hyper-v
    - mac decktop &kitematic :gui container
    - 注意如果下载了win的docker，那么在win上的virtualbox无法使用
    - 建议还是使用虚机
    <!-- more -->
2. [vagrant](https://www.vagrantup.com/)工具
    - vagrant init
    - vagrant up
    - vagrant destory
    - vagrant destory
    - vagrant file
    - vagrant file cloud find[ ubuntu](https://app.vagrantup.com/ubuntu/boxes/trusty64)
    - 可以多台同时执行

3. win上安装vargant安装centos7 
    - ```
       C:\Users\volvo>cd vagrant
        
        C:\Users\volvo\vagrant>ls
        'ls' 不是内部或外部命令，也不是可运行的程序
        或批处理文件。
        
        C:\Users\volvo\vagrant>mkdir centos7
        
        C:\Users\volvo\vagrant>cd centos7
        
        C:\Users\volvo\vagrant\centos7>vagrant init centos/7
        C:\Users\volvo\vagrant\centos7>vagrant init centos/7
        A `Vagrantfile` has been placed in this directory. You are now
        ready to `vagrant up` your first virtual environment! Please read
        the comments in the Vagrantfile as well as documentation on
        `vagrantup.com` for more information on using Vagrant.
        
        C:\Users\volvo\vagrant\centos7>vagrant up
        Bringing machine 'default' up with 'virtualbox' provider...
        
        vagrant ssh

    ```
    
4. 在u2dev虚拟机安装docker的过程
    1. https://www.runoob.com/docker/ubuntu-docker-install.html
5. docker machine，玩云上的，本机的各种docker
6. docker clint & docker server & more docker driver
7. 安装`base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine`
8. docker-machline-driver，可以通过tcp或者unix sock的方式创建和维护d本地和远程docker,可以在官方找到driver，通过access_key 及 access_id来鉴权登录,region可选，通过docker-machine ssh $docker_uanme进入到远程的docker server，可以通过eval命令更改本地的环境主体。
9. docker playground ：https://labs.play-with-docker.com

---
完