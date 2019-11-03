---
title: docker容器技术与简介
date: 2018-06-29 23:07:56
categories: DevOps
tags:
---
1. 容器技术与docker
2. docker能做什么
    - 简化配置（所有打包到容器里）
    - 提升开发效率（环境相同，统一部署）
    - 隔离应用
3. 容器应用代表
<!-- more -->
    - docker and docker swarm
    - docker cloud & docker 企业版（收费）
    - kubernetes k8s（容器编排工具，见下图）
    - ![image](https://note.youdao.com/yws/public/resource/9b29555102c38022c7779af947ee67c2/9D2642E1E9974559BF725E7541BEDAB5?ynotemdtimestamp=1561302581281)
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
    -  ![image](https://note.youdao.com/yws/public/resource/9b29555102c38022c7779af947ee67c2/6FE5AB8ECDBD4169AE3697C6C2A3D2D0?ynotemdtimestamp=1561302581281)

---
1.  容器技术概述
    - 传统模式 硬件不兼容、部署复杂
    - 虚拟机模式 hypervisor，可以实现物理资源的自定义调度资源池
    - 容器技术产生背景（环境各种各样、部署、监控各种各样）（见下图）
    - ![image](https://note.youdao.com/yws/public/resource/9b29555102c38022c7779af947ee67c2/A7D6A7D755DF4CE8AB2339D14C613F30?ynotemdtimestamp=1561302581281)
    - 容器解决了什么问题（解决开发、运维、测试的沟通，见下图）
    - ![image](https://note.youdao.com/yws/public/resource/9b29555102c38022c7779af947ee67c2/B45983FD363F460FA6C00FC6F37197CF?ynotemdtimestamp=1561302934925)
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

---
完
    

