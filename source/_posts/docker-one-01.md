---
title: docker容器技术与简介
date: 2019-01-29 23:07:56
categories: DevOps
tags:
    - docker
---
## 容器技术与docker
## docker能做什么

    - 简化配置（所有打包到容器里）
    - 提升开发效率（环境相同，统一部署）
    - 隔离应用

## 容器应用代表
    - docker and docker swarm
    - docker cloud & docker 企业版（收费）
    - kubernetes k8s（容器编排工具，见下图）
![容器编排][1]
 <!-- more -->
4. devops
    - 文化+过程+工具
    - 持续
        - 集成
        - 发布
        - 测试
        - 监控
        - 改进
    - 自动化
        - 部署
        - 监控
        - 版本管理
    - 信任和尊重、敏捷的目标、开放的沟通
5. 总览（见下图）
    -  ![总览][2]

---
1.  容器技术概述
    - 传统模式 硬件不兼容、部署复杂
    - 虚拟机模式 hypervisor，可以实现物理资源的自定义调度资源池
    - 容器技术产生背景（环境各种各样、部署、监控各种各样）（见下图）
    - ![image][3]
    - 容器解决了什么问题（解决开发、运维、测试的沟通，见下图）
    - ![image][4]
    - 容器定义
        - 对软件和依赖的标准化打包
        - 实现应用之间的隔离
        - 共享同一个os
    - 容器与虚拟机主要区别
        - 容器是app层面的隔离，base os on docker
        - 虚拟化是物理资源层面的隔离
        - 当然两者可以一起使用

---
1. docker魅力（部署wordpress）
    - 依赖
    - build
    - 镜像
    - dockder compose up

---
1. vagrant
2. labs


[1]: /img/DevOps/rongqi_bianpai.png
[2]: /img/DevOps/总览.png
[3]: /img/DevOps/容器技术产生背景.png
[4]: /img/DevOps/容器解决什么问题.png


    

