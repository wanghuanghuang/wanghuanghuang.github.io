---
title: spring-async异步
date: 2019-05-29 11:06:35
tags:
---
# 原理
异步处理适用那些与业务逻辑（横切关注点）不直接相关或者不作为其他业务逻辑输入的部分，也可在分布式系统中解耦。
译注：横切关注点（cross-cutting concerns）指一些具有横越多个模块的行为，使用传统的软件开发方法不能够达到有效模块化的一类特殊关注点。
Spring 中，`@Async`注解可以标记异步操作。然而，使用`@Async`时有一些限制，仅仅把它加在方法上并不能确保方法会在独立的线程中执行。如果你只是偶尔用到 `@Async`，需要格外当心。
## 1.@Async的工作机制
首先为方法添加 `Async` 注解。接着，Spring 会基于 `proxyTargetClass` 属性，为包含 `Async` 定义的对象创建代理（JDK Proxy/CGlib）。最后，Spring 会尝试搜索与当前上下文相关的线程池，把该方法作为独立的执行路径提交。确切地说，Spring 会搜索唯一的 `TaskExecutor` bean 或者名为 `taskExecutor` 的 bean。如果找不到，则使用默认的 `SimpleAsyncTaskExecutor`。
要完成上面的过程，使用中需要注意几个限制，否则会出现 `Async` 不起作用的情况。
默认的SimpleAsyncTasckExecutor 不是真正线程池，他是使用一次，创建一个线程。
## 2.@Async的限制
必须在标记 `@ComponentScan` 或 `@configuration` 的类中使用 `@Async`。
## 3.@Async的使用
### 3.1 @Async的使用类
```
@Component
public class AsyncExportTrigger {    
    @Async
    public void exportVehiclesCsv() {
       
    }
}
```
### 3.2 调用@Async的使用类
```
@RestController
public class VehicleController {
    @Autowired
    private AsyncExportTrigger trigger;
    
    public ResponseMessage exportVehiclesCsv() {
        trigger.exportVehiclesCsv();
    }	
}
```
上面的例子中，使用了 `@Autowired` 的 `AsyncExportTrigger` 受 `@Component` 管理，因而会创建新线程执行。如果不是注入`AsyncExportTrigger` 而是`new`的话，是创建局部对象，不受管理，不会创建新的线程。
注：`@RestController` 是一个联合注解，包含`@Component`注解。
#### 注意
（1）不要在`private`方法上使用`@Async`注解。由于在运行时不能创建代理，所以不起作用。
（2）调用标有`@Async` 的方法的方法与标有`@Async`的方法要在不同的类中定义。
（3）执行的时候，要在启动类上加`@EnableAsync`注解。它的作用是让Spring在后台线程池中提交`@Async`方法。
（4）使用async时，不要直接传 `request` 对象或任何与`Request/Response`相关的对象，可以创建一个`value` 对象，将对象设置信息后，将对象传递给Spring Async。

捕捉异常 参考地址：
<https://mp.weixin.qq.com/s?__biz=MjM5NzMyMjAwMA==&mid=2651483391&idx=2&sn=91f2c441793c0d70599ad92671201b75&chksm=bd2500808a5289965107f0cc6d8cf75f850a188daf62d7c4330585e1f604f4398749cdf0d26b&scene=21#wechat_redirect>

