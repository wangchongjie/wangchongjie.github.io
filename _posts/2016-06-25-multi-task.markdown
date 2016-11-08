---
layout:     post
title:      "Java Open-Source framework MultiTask"
subtitle:   " \"Quick Start & Manual.\""
date:       2016-11-01 22:55:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 开源框架
---

# multi-task
Multi-task is a Java framework for parallel processing which is based annotation. That is high performance, not intrusive and loose coupled.

## 1. Quick Start
This chapter will show you how to get started with Multi-Task.
[中文使用说明](https://github.com/wangchongjie/multi-engine/wiki)

### 1.1 Prerequisite
In order to use Multi-Task within a Maven project, simply add the following dependency to your pom.xml. 

```
	<dependency>
    	<groupId>com.baidu.unbiz</groupId>
    	<artifactId>multi-task</artifactId>
    	<version>1.0.0</version>
	</dependency>
```

### 1.2 Create a normal service with annotation
Create a normal service class whose methods will be called parallely on later. For example, a DevicePlanStatServiceImpl class is created as below.

```
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
The class is marked by `@TaskService`, which could be scanned by Multi-Task framework. The `@TaskBean(task name)` is attached on the method. Then, the method could be regarded as parallel task. 

### 1.3 Applying parallely processing with defined task

```
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
The task deviceStatFetcher and deviceUvFetcher will be parallely processing and atomic return. Actually, the method queryPlanDeviceData and queryPlanDeviceUvData of class DevicePlanStatServiceImpl will be implicitly executed.

### 1.4 Some other type' TaskBean
Besides single param method, we could also define multi-param or void param' method for task by using `@TaskBean`.

```
@TaskService
public class OtherStatServiceImpl implements OtherStatService {
   
    @TaskBean("multiParamFetcher")
    public List<DeviceViewItem> queryPlanDeviceDataByMultiParam(String p1, int p2, int p3) {
        return this.mockList1();
    }

    @TaskBean("voidParamFetcher")
    public List<DeviceViewItem> queryPlanDeviceDataByVoidParam() {
        return this.mockList2();
    }
    
    @TaskBean("doSthFailWithExceptionFetcher")
    public List<DeviceViewItem> queryPlanDeviceDataWithBusinessException(DeviceRequest req) {
        throw new BusinessException("Some business com.baidu.unbiz.multitask.vo.exception, just for test!");
    }
    
    @TaskBean("doSthVerySlowFetcher")
    public List<DeviceViewItem> queryPlanDeviceDataWithBadNetwork(DeviceRequest req) {
        try {
            Thread.sleep(900000L);
        } catch (InterruptedException e) {
            // do nothing, just for test
        }
        return this.mockList1();
    }
}
```

## 2. Advanced features

### 2.1 Explicit define task
You can also define Task by implement `Taskable<T>` interface. But, not recommended.

```
@Service
public class ExplicitDefTask implements Taskable<List<DeviceViewItem>> {

    public <E> List<DeviceViewItem> work(E request) {
        if (request instanceof  DeviceRequest) {
            // do sth;
            return result;
        }
        return null;
    }
}
```

### 2.2 Explicit config thread pool
Multi-Task will specify the thread pool configuration automatically considering hardware resources. It is possible to set pool parameters explicitly as well. Examples are as follows.

```
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

### 2.3 Fork Join
Multi-task could handle homogenous computing more friendly by fork-join mode. ForkJoin strategy should be provided to framework.

```
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

### 2.4 ThreadLocal Support
Multi-task will ingore ThreadLocal of Java by default. If you want to let the ThreadLocal take effect in running task, you should do the following configuration once:

```
    TaskContext.attachThreadLocal(MyThreadLocal.instance());
```    
    
## 3. Examples
All test cases or samples can be found from the below links:

[Samples - How to define task](https://github.com/wangchongjie/multi-task/tree/master/src/test/java/com/baidu/unbiz/multitask/service)

[Test cases - How to use task](https://github.com/wangchongjie/multi-task/tree/master/src/test/java/com/baidu/unbiz/multitask/demo/test)

[Resources - Optional, not necessary](https://github.com/wangchongjie/multi-task/tree/master/src/test/resources)

This project is licensed under [Apache v2 license](http://www.apache.org/licenses/LICENSE-2.0.txt).
