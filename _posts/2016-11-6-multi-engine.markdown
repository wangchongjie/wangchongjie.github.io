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
项目实践中，我们经常会遇到单机并行或分布式并行可以大幅提升系统整体性能的场景。但由于受限于线程模型、锁机制等相对较为复杂，并行化改造的成本较高。multi-engine系列组件提供轻量级的开箱即用的并行化支持特性，并且单机模型已在生产环境得到大量应用，并带来可观的性能收益。本文介绍并行计算multi-engine系列组件的使用说明，源码已托管于github，且稳定版本已发布至Maven中央仓库，可直接使用。本文目录如下：

# 目录
1. Multi-Engine介绍 
1.1 multi-engine是什么    
1.2 multi-engine功能概述   
2. 设计架构   
3. 使用方式   
3.1 准备工作   
3.2 应用方业务开发    
3.3 最佳实践   
4. 程序测试   
4.1 测试考虑   
5. 改进升级   
6. 附录 
6.1 生产环境实战效果   
6.2 Amdahl加速定律 


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
服务应用方需要依赖multi-task、multi-engine、cluster-support，该模块见github的相关目录：

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
通过mvn dependency:tree命令分析，multi-task依赖以下三方库：
[INFO] com.baidu.unbiz:multi-task:jar:1.0.1
[INFO] +- commons-lang:commons-lang:jar:2.4:compile
[INFO] +- org.springframework:spring-context:jar:4.1.7.RELEASE:compile
[INFO] |  +- org.springframework:spring-aop:jar:4.1.7.RELEASE:compile
[INFO] |  |  \- aopalliance:aopalliance:jar:1.0:compile
[INFO] |  \- org.springframework:spring-expression:jar:4.1.7.RELEASE:compile
[INFO] +- org.springframework:spring-test:jar:4.1.7.RELEASE:test
[INFO] +- org.springframework:spring-beans:jar:4.1.7.RELEASE:compile
[INFO] +- org.springframework:spring-core:jar:4.1.7.RELEASE:compile
[INFO] |  \- commons-logging:commons-logging:jar:1.2:compile
[INFO] +- ch.qos.logback:logback-core:jar:1.0.0:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.0.0:compile
[INFO] |  \- org.slf4j:slf4j-api:jar:1.6.4:compile
[INFO] +- junit:junit:jar:4.11:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.jmock:jmock-junit4:jar:2.5.1:test
[INFO]    \- org.jmock:jmock:jar:2.5.1:test
[INFO]       \- org.hamcrest:hamcrest-library:jar:1.1:test

Multi-engine、cluster-support的依赖Jar不一一列举，可以mvn dependency:tree命令分析。
## 3.1.2 配置线程资源
Multi-task如无特殊需求，可以零配置，组件会根据系统环境自动设置相关变量。如需自定义指定，可参考配置如下：
```xml
 <bean name="xmlThreadPoolConfig" class="com.baidu.unbiz.multitask.constants.XmlThreadPoolConfig">
     <property name="coreTaskNum" value="12"/>
     <property name="maxTaskNum" value="22"/>
     <property name="maxCacheTaskNum" value="4"/>
     <property name="queueFullSleepTime" value="10"/>
     <property name="taskTimeoutMillSeconds" value="5000"/>
 </bean>

 <bean name="simpleParallelExePool" class="com.baidu.unbiz.multitask.task.SimpleParallelExePool">
     <constructor-arg ref="xmlThreadPoolConfig"/>
 </bean>
```
## 3.1.3 配置服务地址及本地服务端口
若选用multi-engine模块，需配置task服务地址及本机暴露的接口，相关配置如下：
```xml
<bean name="endpointSupervisor" class="com.baidu.unbiz.multiengine.endpoint.supervisor.DefaultEndpointSupervisor"
      init-method="init" destroy-method="stop">
    <property name="serverHost" value="127.0.0.1:8801;127.0.0.2:8802"/>
    <property name="exportPort" value="8801"/>
</bean>
```
其中，serverHost为集群所有机器的ip和端口（含本机）。exportPort为本机器实例对外暴露的接口。缺省的终端管理DefaultEndpointSupervisor是通过heartbeat和gossip同步集群信息的。
## 3.1.4 Cluster-support配置
若选用cluster-support模块，需配置ClusterEndpointSupervisor和zoo.cfg，相关配置如下：
```xml
   <bean name="endpointSupervisor" class="com.baidu.unbiz.multiengine.cluster.endpoint.supervisor.ClusterEndpointSupervisor"
      init-method="init" destroy-method="stop">
    <property name="exportPort" value="8801;8802"/>
</bean>
```

tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper
clientPort=8701
clientPortAddress=xx.xx.xx.xx
server.1=localhost:2888:3888

## 3.1.5 配置logback
Multi-engine推荐使用logback作为日志组件，在logback.xml中加入如下配置，默认level=”INFO”，如想打开multi-engine的debug模式可以设置level=”DEBUG”。
```xml
   <logger name="com.baidu.unbiz.multiengine" level="INFO" additivity="true">
   </logger> 
```
# 3.2 应用方业务开发
应用方demo代码，可以参考各模块的test目录：com.baidu.unbiz.*.demo.test。
## 3.2.1 service上通过注解定义task
1、在普通service类上打@TaskService注解，在需要定义为并行组件的方法上打@TaskBean注解，其中属性为task的名称。即完成并行task的定义。
```java
@TaskService
public class DevicePlanStatServiceImpl implements DevicePlanStatService {
  @TaskBean("deviceStatFetcher")
  public List<DeviceStatViewItem> queryPlanDeviceData(DeviceStatRequest req) {
        this.checkParam(req);
        return this.mockList1();
    }

  @TaskBean("deviceUvFetcher")
  public List<DeviceUvViewItem> queryPlanDeviceUvData(DeviceUvRequest req) {
        this.checkParam(req);
        return this.mockList2();
    }
}
```
2、除注解定义方式外，还可以显示通过实现接口的方式定义并行task。
```java
@Service
public class ExplicitDefTask 
    implements Taskable<List<DeviceViewItem>> {

    public <E> List<DeviceViewItem> work(E request) {
        if (request instanceof  DeviceRequest) {
            // do sth;
            return result;
        }
        return null;
    }
}
```
## 3.2.2 应用定义的task并行处理
并行task定义完成后，上层应用代码示例如下：
```java
 @Resource(name = "simpleParallelExePool")
    private ParallelExePool parallelExePool;

    public void testParallelFetch() {
        DeviceStatRequest req1 = new DeviceStatRequest();
        DeviceUvRequest req2 = new DeviceUvRequest();

        MultiResult ctx = parallelExePool.submit(
                new TaskPair("deviceStatFetcher", req1),
                new TaskPair("deviceUvFetcher", req2));

     List<DeviceStatViewItem> stat = ctx.getResult("deviceStatFetcher");
     List<DeviceUvViewItem> uv = ctx.getResult("deviceUvFetcher");

        Assert.notEmpty(stat);
        Assert.notEmpty(uv);
    }
```
此处，deviceStatFetcher、deviceUvFetcher两个task会并行执行，本质上是隐式执行了DevicePlanStatServiceImpl的被@TaskBean注解的两个方法。
## 3.2.3 Fork-Join模式支持
对于同构计算，multi-task 、multi-engine非常友好地支持了Fork-Join，示例代码如下：
```java
public void testParallelForkJoinFetch() {
    TaskPair taskPair = new TaskPair("deviceStatFetcher", new DeviceRequest()));

    ForkJoin<DeviceRequest, List<DeviceViewItem>> forkJoin = new ForkJoin<DeviceRequest, List<DeviceViewItem>>() {

    public List<DeviceRequest> fork(DeviceRequest deviceRequest) {
            List<DeviceRequest> reqs = new ArrayList<DeviceRequest>();
            reqs.add(deviceRequest);
            reqs.add(deviceRequest);
            reqs.add(deviceRequest);
            return reqs;
        }

    public List<DeviceViewItem> join(List<List<DeviceViewItem>> lists) {
            List<DeviceViewItem> result = new ArrayList<DeviceViewItem>();
            if (CollectionUtils.isEmpty(lists)) {
                return result;
            }
            for (List<DeviceViewItem> res : lists) {
                result.addAll(res);
            }
            return result;
        }
    };

    List<DeviceViewItem> result = parallelExePool.submit(taskPair, forkJoin);
    Assert.notEmpty(result);
}
```
## 3.2.4 ThreadLocal支持
Multi-task的执行task缺省情况下会忽略ThreadLocal，如果需要用ThreadLocal进行参数传递，可以做如下配置：
```java
TaskContext.attachThreadLocal(MyThreadLocal.instance());
```
## 3.2.5分布式Task执行
以上示例代码均是单机多线程执行，若要分布式多进程执行，只需用new DisTaskPair()替代new TaskPair()即可。单机与分布式共享一套API，并且可以混合执行（既有TaskPair又有DisTaskPair）。 
```java
    @Resource(name = "distributedParallelExePool")
    private ParallelExePool parallelExePool;

    public void testParallelFetch() {
        DeviceStatRequest req1 = new DeviceStatRequest();
        DeviceUvRequest req2 = new DeviceUvRequest();

        MultiResult ctx = parallelExePool.submit(
                new DisTaskPair("deviceStatFetcher", req1),
                new DisTaskPair("deviceUvFetcher", req2));

     List<DeviceStatViewItem> stat = ctx.getResult("deviceStatFetcher");
     List<DeviceUvViewItem> uv = ctx.getResult("deviceUvFetcher");

        Assert.notEmpty(stat);
        Assert.notEmpty(uv);
    }
```
# 3.3 最佳实践
* 1)   业务场景为单机并行处理，则只引入multi-task组件。
* 2)   业务场景为分布式并行处理，则建议multi-task、multi-engine、cluster-support一起使用。
* 3)   无Zookeeper的基础设施，但需分布式并行处理，则建议使用multi-task、multi-engine。

# 4.   程序测试
对multi-task、multi-engine、cluster-support三个组件进行独立或组装测试。
## 4.1 测试考虑
测试代码可参考对应组件的test目录，有相应的测试case。已对多线程或多进程的同构或异构计算，集群支持等进行了较为充分的测试。

*  性能：组件提供并行处理支持，串行改并行后理论上性能会有明显提升。
*  伸缩性：支持集群横向扩展，弹性伸缩。
*  扩展性：框架预留扩展接口，支持自定义功能扩展。
*  可用性（HA）：框架保障元信息同步或服务发现、健康检测等，内部实现HA切换。

# 5.   改进升级
* 1、   硬件资源探测及反馈。
* 2、   更多计算模型的支持。
* 3、   其它…

# 6.   附录
## 6.1 生产环境实战效果
线上生产环境的cpu核数较多，以线上某报表服务的机器为例，有24个处理器：

该报表请求会查4份数据，分别耗时：18ms、28ms、29ms、31ms，总时间为106ms。但应用并行化组件multi-task后，仅耗时32ms即完成了该次请求。

## 6.2 Amdahl加速定律
S = 1 / ( 1 – a + a / n )
S:  并行处理效果的加速比
a:   并行计算部分所占比例
n:   并行处理结点个数
最小加速比s=1
n→∞,极限加速比为1/（1-a）

例：若串行代码占整个代码的25%，则并行处理的总体性能不可能超过4。