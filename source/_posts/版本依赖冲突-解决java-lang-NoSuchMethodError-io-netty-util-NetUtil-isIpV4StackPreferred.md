---
title: >-
  版本依赖冲突-解决java.lang.NoSuchMethodError:io.netty.util.NetUtil.isIpV4StackPreferred()
date: 2020-01-19 16:36:18
tags:
---

在开发中，用到redis分布式锁，就引入了自己公司开发封装的jar包，jar包内引用了最新的3.12.0版本的redission，导致项目启动时报错
![报错图片1](/images/版本依赖冲突-解决java.lang.NoSuchMethodError/isIpV4StackPreferred-1.png)
错误信息打印输出：
```
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'lockFactory': Unsatisfied dependency expressed through field 'redissonClient'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'redisson' defined in class path resource [com/diditech/iov/kernel/dlock/DlockAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.redisson.api.RedissonClient]: Factory method 'redisson' threw exception; nested exception is java.lang.NoSuchMethodError: io.netty.util.NetUtil.isIpV4StackPreferred()Z
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:588)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:366)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1225)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:552)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:207)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1136)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1064)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:585)
	... 19 common frames omitted
```
看到是因为没有找到校验IPV4或v6的方法报错，复制isIpV4StackPreferred在idea中查找哪里使用
![报错图片2](/images/版本依赖冲突-解决java.lang.NoSuchMethodError/isIpV4StackPreferred-2.png)

查找到引用类，再定位到所在的依赖包
![报错图片3](/images/版本依赖冲突-解决java.lang.NoSuchMethodError/isIpV4StackPreferred-3.png)

是netty-all报错，肯定是引入新jar后，出现依赖问题，我们就去看看这个出错jar在哪里有引用
![报错图片4](/images/版本依赖冲突-解决java.lang.NoSuchMethodError/isIpV4StackPreferred-4.png)

在maven-helper帮助，搜索发现是core包中引用的kernel-cat，依赖中包含引用了4.0.24版本的netty-all，此版本中没有使用isIpV4StackPreferred()
故，在引入core时，去除引入netty-all依赖,保存设置就解决了
![报错图片5](/images/版本依赖冲突-解决java.lang.NoSuchMethodError/isIpV4StackPreferred-5.png)

### 结论：
发现 公司封装的redis分布式锁依赖中引入的是netty-resolver-dns : 4.1.44.Final版本，其中用到netty-all中方法验证IPV4方法，需要使用netty-all 4.1.44版本，报错是因为使用了低版本的netty-all




