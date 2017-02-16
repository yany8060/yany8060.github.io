---
title: Yarn 概述
date: 2017-02-16 20:15:59
tags: [yarn,hadoop]
categories: [Big data]
---
### 名词介绍
ResourceManager：简称RM，是YARN资源控制框架的中心模块，负责集群中所有的资源的统一管理和分配，它接收来自NM（NodeManager）的汇报，建立AM，并将资源派送给AM（ApplicationMaster）。
NodeManager：简称NM，NodeManager是ResourceManager在每台机器上的代理，负责容器的管理，并监控他们的资源使用情况（CPU、内存、磁盘及网络等），以及向ResourceManager提供这些资源使用报告。
ApplicationMaster：简称AM，YARN中每个应用都会启动一个AM，负责向RM申请资源，请求NM启动container，并告诉container要做什么事情。
Container：资源容器。YARN中所有的应用都是在container之上运行的。AM也是在container上运行的，不过AM的container是RM申请的。
Apache Hadoop YARN（Yet Another Resource Negotiator，另一个资源协调者）：是一种新的Hadoop资源管理器，它是一个通用资源管理系统。

![yarn.png](http://upload-images.jianshu.io/upload_images/1419542-0aad7c3fd732d90a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Yarn主要架构
#### ResourceManager（RM）
ResourceManager是一个全局的资源管理器，负责整个系统的资源管理和分配。它主要由两个组件构成： 调度器 （Scheduler）和 应用程序管理器 （Applications Manager，ASM）。
* 调度器（Scheduler）：调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。
* 应用程序管理器：应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等。

####  ApplicationMaster（AM）
用户提交的每个应用程序均包含一个AM，主要功能包括：
* 与RM调度器协商以获取资源（以Container表示）
* 将得到的任务进一步分配给内部的任务
* 与NM通信以启动/停止任务
* 监控所有任务运行状态，并在任务失败时重新为任务申请资源以重启任务

#### NodeManager（NM）
NM是每个节点上的资源和任务管理器。
* 它定时地向RM汇报本节点的资源使用情况和Container运行状态；
* 它接受并处理来自AM的Container启动/停止等各种请求。

#### Container
Container是YARN中的资源抽象，它封装了某个节点上的多维资源，如CPU、内存、磁盘、网络等。当AM向RM申请资源时，RM向AM返回的资源便是用Container表示的。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源。Container是一个动态资源划分单位，是根据应用程序的需求自动生成的。目前，YARN仅支持CPU和内存两种资源。

### YARN工作流程
![yarn-progress-20170216.png](http://upload-images.jianshu.io/upload_images/1419542-4577d88af297c5f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1、用户向YARN中提交应用程序，其中包括ApplicationMaster程序、启动ApplicationMaster的命令、用户程序等。
2、ResourceManager为该应用程序分配第一个Container，并与对应的NodeManager通信，要求它在这个Container中启动应用程序的ApplicationMaster。
3、ApplicationMaster首先向ResourceManager注册，这样用户就可以直接通过ResourceManager查看应用程序的运行状态，然后它将为各个任务申请资源，并监控它的运行状态，直到运行结束，即重复步骤4~7。
4、ApplicationMaster采用轮询的方式通过RPC协议向ResourceManager申请和领取资源。
5、一旦ApplicationMaster申请到资源后，便与对应的NodeManager通信，要求它启动任务。
6、NodeManager为任务设置好运行环境（包括环境变量、JAR包、二进制程序等）后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务。
7、各个任务通过某个RPC协议向ApplicationMaster汇报自己的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。
8、应用程序运行完成后，ApplicationMaster向ResourceManager注销并关闭自己。