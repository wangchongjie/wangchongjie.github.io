---
layout:     post
title:      "并行计算组件MultiEngine框架"
subtitle:   " \"之自研框架Manual.\""
date:       2016-11-06 22:55:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 开源框架
---
项目实践中，我们经常会遇到单机并行或分布式并行可以大幅提升系统整体性能的场景。但由于受限于线程模型、锁机制等相对较为复杂，并行化改造的成本较高。multi-engine系列组件提供轻量级的开箱即用的并行化支持特性，并且单机模型已在生产环境得到大量应用，并带来可观的性能收益。本文介绍并行计算multi-engine系列组件的使用说明，源码已托管于github，且稳定版本已发布至Maven中央仓库，可直接使用。

# 1.   Multi-Engine介绍
## 1.1 multi-engine是什么
Multi-engine是分布式多任务并行处理的基础组件：可通过Java注解对原有业务代码几乎无侵入地实现并行化，由multi-task、multi-engine、cluster-support三个独立可插拔的组件组成。各组件一起组合使用，也可根据所需feature独立使用其中的一两个组件。

该组件设计初衷是：为传统业务代码提供单机或集群并行处理的利器。应用方无需关心线程、锁、资源、通信、分布式故障等问题。而将工作重心更多地关注业务开发即可。遵循Amdahl加速定律，改善提升系统的处理效率，降低响应延迟。

Multi-engine可理解为与业务无关的单机或分布式计算模型封装，为“多任务处理引擎”。是一个轻量级的并行处理组件。重点应用场景为传统web或cron代码的单机或集群化并行计算，不解决T级别海量数据处理或弹性分布式数据集等问题（后者有更好的工具支持，如spark、hadoop等）。
## 1.2 multi-engine功能概述
1、multi-task组件
Multi-task组件为基础组件，提供单机多线程的并行处理模型，并预留用户自定义扩展接口。该组件封装了task定义的方式（接口或注解）、单机同构并行计算或异构计算等。作为容器持有了用户标注的可并行处理的task。该组件可独立使用，提供单机多任务处理的能力。

2、multi-engine组件
Multi-engine组件是multi-task的功能扩展，为分布式版的multi-task组件。组件接口及计算模型与multi-task一致，应用时选用对应的并行池化执行组件即可。该组件结合multi-task组件，开箱即用：实现分布式并行处理（无需其它协调设施或db存储）。组件通过heartbeat和gossip协议，sync集群信息。

3、cluster-support组件
Cluster-support组件是multi-engine的功能扩展，为multi-engine提供第三方元信息管理支持。可替代multi-engine原生的gossip信息同步。Cluster-support默认的分布式元信息管理是由Zookeeper实现的，用户也可自定义其它实现方式。以上3个组件一起使用时，需配置元信息管理的Zookeeper集群。 

以上3个组件为预实现的组件，设计思路为可插拔、插件化支持。用户也可根据需求扩展已有组件。
# 2.   设计架构
广义上的Multi-Engine采用插件化的设计，由multi-task、multi-engine、cluster-support三个组件构成。各组件对应主要职责划分如下：

以上组件有3种使用方式：
*  Only multi-task：实现单机多线程版本的多任务并行处理。
*  Multi-task + Multi-engine: 无需其它设施，实现分布式多任务并行处理。
*  Multi-task + Multi-engine + Cluster-support: 实现Zookeeper方式协调的分布式多任务并行处理，需提供Zookeeper集群。

通信协议层，组件为了尽量减少网络开销，降低协议头负载，自定义了一套字节传输协议，packHead+protostuff/protobuf/json。同时也支持NsHead协议，或扩展定制其它协议。

框架的设计初衷是，尽量不改变用户的编程习惯（少侵入），使得用户轻松开发并行化的业务代码，提升改善系统的性能。
# 3.   使用方式
为了快速了解multi-engine如何使用，我们来做一个简单的hello world。
## 3.1 准备工作
服务应用方需要依赖multi-task、multi-engine、cluster-support，该模块见github的相关目录.

## 3.1.1 配置maven
业务应用方需要依赖multi-task、multi-engine、cluster-support等模块（按需所取），该模块用maven进行源代码管理，在pom.xml中加入dependency： 
```xml
      <dependency>
         <groupId>com.baidu.unbiz</groupId>
         <artifactId>multi-task</artifactId>
         <version>1.0.1</version>
      </dependency>
      <dependency>
         <groupId>com.baidu.unbiz</groupId>
         <artifactId>multi-engine</artifactId>
         <version>1.0.0</version>
      </dependency>
      <dependency>
         <groupId>com.baidu.unbiz</groupId>
         <artifactId>cluster-support</artifactId>
         <version>1.0.0</version>
      </dependency>
```