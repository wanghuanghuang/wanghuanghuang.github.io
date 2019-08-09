---
title: lambda中flatMap与map用法
date: 2019-08-07 10:20:09
tags:
---
## map
流中的map方法，可以接受一个函数作为参数，这个函数会被应用到每个元素上，并将其映射成一个新的元素。
### 示例：
给定一个单词列表，你想要返回另一个列表，显示每个单词中有几个字母。
### 代码示例
```
//单词列表
List<String> words = Arrays.asList("java8","lambda","action");
List<Integer> wordLength = words.stream()
    .map(String::length)
    .collect(toList());
```
## flatMap
flatMap方法可以把一个流中的每个值换成另一个流，然后把所有的流连接起来形成一个流，即扁平化一个流。
### 示例：
给定一张单词表，返回表里各不相同的字母。
### 代码示例
```
List<String> uniqueCharacters = words.stream()
            .map(w -> w.split(""))//将每个单词转换为由其字母构成的数组
            .flatMap(Arrays::stream).distinct() //将各个生成扁平流化为单个流
            .collect(Collectors.toList()); 
```
示例解析图：
![flatMap解析图](/images/lambda中flatMap与map用法/1.png)
### flatMap应用于map,list 代码示例：
```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Java8FlatMap {

    public static void main(String[] args) {
        // 1.flat list 
        List<List<Integer>> list = new ArrayList<>();
        List<Integer> subList1 = new ArrayList<Integer>(){{add(1);add(2);add(3);add(4);}};
        List<Integer> subList2 = new ArrayList<Integer>(){{add(1);add(2);add(3);add(4);}};
        list.add(subList1);
        list.add(subList2);

        BigDecimal result = list.stream()
                //.flatMap(sublist -> sublist.stream())
                .flatMap(List::stream)
                .map(i -> new BigDecimal(i))
                .reduce(BigDecimal.ZERO , (x, y) -> x.add(y));
        System.out.println(result);

        // 2.flat map
        Map<String, Map<String, BigDecimal>> testMAP = new HashMap<>();
        Map<String, BigDecimal> subMap1 = new HashMap<>();
        subMap1.put("1", new BigDecimal(1));
        testMAP.put("1", subMap1);
        Map<String, BigDecimal> subMap2 = new HashMap<>();
        subMap2.put("2", new BigDecimal(2));
        testMAP.put("2", subMap2);
        BigDecimal result2 = testMAP.values().stream()
                .map(Map::entrySet)
                .flatMap(Collection::stream)
                .map(subMap -> subMap.getValue())
                .reduce(BigDecimal.ZERO , (x, y) -> x.add(y));
        System.out.println(result2);
    }

}

```