---
title: copyProperties性能比较
date: 2019-06-12 14:55:55
tags:
---
## 一.原因
阿里编程规范提示"避免使用Apache BeanUtils.copyProperties()方法"。

## 二.copy工具
1.springframework的BeanUtils
2.cglib的BeanCopier
3.Apache BeanUtils包的PropertyUtils类

## 三.代码测试性能

### 1）StaticCglibBeanCopierPropertiesCopier
```
 package com.diditech.iov.ZropertiesCopier;

import org.assertj.core.internal.cglib.beans.BeanCopier;

// 全局静态 BeanCopier，避免每次都生成新的对象
public class StaticCglibBeanCopierPropertiesCopier implements PropertiesCopier {
    private static BeanCopier copier = BeanCopier.create(Account.class,Account.class, false);

    @Override
    public void copyProperties(Object source,Object target) throws Exception {
        copier.copy(source, target, null);
    }
}
```
### 2）CglibBeanCopierPropertiesCopier
```
package com.diditech.iov.ZropertiesCopier;

import org.assertj.core.internal.cglib.beans.BeanCopier;

public class CglibBeanCopierPropertiesCopier implements PropertiesCopier {

    @Override
    public void copyProperties(Object source, Object target) throws Exception {
        BeanCopier copier = BeanCopier.create(source.getClass(), target.getClass(), false);
        copier.copy(source, target, null);
    }
}
```
### 3）SpringBeanUtilsPropertiesCopier
```
package com.diditech.iov.ZropertiesCopier;

public class SpringBeanUtilsPropertiesCopier implements PropertiesCopier {

    @Override
    public void copyProperties(Object source, Object target) throws Exception {
        org.springframework.beans.BeanUtils.copyProperties(source, target);
    }
}
```
### 4）CommonsPropertyUtilsPropertiesCopier
```
package com.diditech.iov.ZropertiesCopier;

public class CommonsPropertyUtilsPropertiesCopier implements PropertiesCopier {

    @Override
    public void copyProperties(Object source, Object target) throws Exception {
        org.apache.commons.beanutils.PropertyUtils.copyProperties(target, source);
    }
}
```
### 5）CommonsBeanUtilsPropertiesCopier
```
package com.diditech.iov.ZropertiesCopier;

public class CommonsBeanUtilsPropertiesCopier implements PropertiesCopier {

    @Override
    public void copyProperties(Object source, Object target) throws Exception {
        org.apache.commons.beanutils.BeanUtils.copyProperties(target, source);
    }
}
```
### 6）测试代码
```
package test;

import com.diditech.iov.ZropertiesCopier.Account;
import com.diditech.iov.ZropertiesCopier.CglibBeanCopierPropertiesCopier;
import com.diditech.iov.ZropertiesCopier.CommonsBeanUtilsPropertiesCopier;
import com.diditech.iov.ZropertiesCopier.CommonsPropertyUtilsPropertiesCopier;
import com.diditech.iov.ZropertiesCopier.PropertiesCopier;
import com.diditech.iov.ZropertiesCopier.SpringBeanUtilsPropertiesCopier;
import com.diditech.iov.ZropertiesCopier.StaticCglibBeanCopierPropertiesCopier;
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;

@RunWith(Parameterized.class)
public class PropertiesCopierTest {

    @Parameterized.Parameter(0)
    public PropertiesCopier propertiesCopier;

    // 测试次数
    private static List<Integer> testTimes =
            Arrays.asList(100,1000,10_000,100_000,1_000_000);

    // 测试结果以 markdown 表格的形式输出
    private static StringBuilder resultBuilder = new StringBuilder("|实现|100|1,000|10,000|100,000|1,000,000|\n")
            .append("|----|----|----|----|----|----|\n");


    @Parameterized.Parameters
    public static Collection<Object[]> data() {
        Collection<Object[]> params = new ArrayList<>();
        params.add(new Object[]{new StaticCglibBeanCopierPropertiesCopier()});
        params.add(new Object[]{new CglibBeanCopierPropertiesCopier()});
        params.add(new Object[]{new SpringBeanUtilsPropertiesCopier()});
        params.add(new Object[]{new CommonsPropertyUtilsPropertiesCopier()});
        params.add(new Object[]{new CommonsBeanUtilsPropertiesCopier()});
        return params;
    }


    @Before
    public void setUp() throws Exception {
        String name = propertiesCopier.getClass().getSimpleName()
                .replace("PropertiesCopier","");
        resultBuilder.append("|")
                .append(name)
                .append("|");
    }


    @Test
    public void copyProperties() throws Exception {
        Account source = new Account(1, "test1", 30D);

        Account target = new Account();

        // 预热一次
        propertiesCopier.copyProperties(source, target);
        for (Integer time : testTimes) {
            long start = System.nanoTime();
            for (int i = 0;i < time; i++) {
                propertiesCopier.copyProperties(source, target);
            }
            resultBuilder.append((System.nanoTime() - start) / 1_000_000D)
                    .append("|");
        }
        resultBuilder.append("\n");
    }


    @AfterClass
    public static void tearDown() throws Exception {
        System.out.println("测试结果：");
        System.out.println(resultBuilder);
    }

}
```

## 四.测试结果
测试结果：
|      实现            |   100   |  1,000  |  10,000  |  100,000 | 1,000,000 |
|---------------------|---------|---------|----------|----------|-----------|
|StaticCglibBeanCopier|0.009898 |0.133792 |0.785009  |2.500084  |7.311507   |
|CglibBeanCopier      |1.210962 |5.471856 |16.834693 |57.636399 |127.469792 |
|SpringBeanUtils      |0.077135 |0.831427 |3.784427  |9.591105  |98.63265   |
|CommonsPropertyUtils |0.84781  |8.647047 |27.575667 |42.155676 |340.311382 |
|CommonsBeanUtils     |24.979334|48.169871|342.472206|720.208048|5117.095726|


## 五.结论
1.Apache BeanUtils 性能最差，不建议使用。
2.Apache PropertyUtils 100000次以内性能还能接受，到百万级别性能就比较差了。
3.SpringBeanUtils和CglibBeanCopier性能比较好。

## 六.Apache BeanUtils.copyProperties()性能分析
执行1000000次copy属性,然后通过jvisualvm查看方法耗时,如图:
![Apache BeanUtils.copyProperties()耗时](/images/copyProperties性能比较/11.png)
发现最耗时的方法就是method.invoke(),但是spring的BeanUtils、PropertyUtils里也是采用反射来实现的,为什么效率相差这么大呢？
![结果](/images/copyProperties性能比较/12.png)
看来Abstract.convert()和getIntrospectionData()占用了很大一部分时间.而且Apache BeanUtils中的日志输出也比较耗时。
## 七.Spring BeanUtils和PropertyUtils 性能分析
![Spring BeanUtils和PropertyUtils 性能分析](/images/copyProperties性能比较/13.png)
PropertyUtils和Apache BeanUtils核心代码区别在图中标注的地方。Apache BeanUtils主要集中了各种丰富的功能(日志、转换、解析等等),导致性能变差。

而Spring BeanUtils则是直接通过反射来读取和写入,直抒胸臆,省去了其他繁杂的步骤,性能自然不差。
![Spring BeanUtils](/images/copyProperties性能比较/14.png)

## 八.Cglib BeanCopier
cglib BeanCopier的主要耗时方法就在BeanCopier.create(),如果将该方法做成静态成员变量,则还可以大大缩小执行时间。BeanCopier是一种基于字节码的方式,其实就是通过字节码方式转换成性能最好的get、set方式,只需考虑创建BeanCopier的开销,如果我们将BeanCopier做成静态的,基本只需考虑get、set的开销,所以性能接近于get、set。
BeanCopier源码分析可见:
[BeanCopier源码分析](http://www.jianshu.com/p/f8b892e08d26)

