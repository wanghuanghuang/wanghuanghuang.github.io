---
title: optional用法
date: 2019-08-06 17:07:57
tags:
---
## 简介：
Optional类是一个可以为null的对象，如果存在则调用isPresent()方法会返回true，调用get()方法会返回对象。
Optional是一个容器可以保存类型T值，或者仅仅保存null值。
Optional提供很多方法，这样我们可以不用显式进行空值检测。很好的解决空指针异常。
## Optional方法：
1.   static <T> Optional<T> empty()
    返回空的 Optional 实例。

2.	boolean equals(Object obj)
    判断其他对象是否等于 Optional。

3.	Optional<T> filter(Predicate<? super <T> predicate)
    如果值存在，并且这个值匹配给定的 predicate，返回一个Optional用以描述这个值，否则返回一个空的Optional。

4.	<U> Optional<U> flatMap(Function<? super T,Optional<U>> mapper)
    如果值存在，返回基于Optional包含的映射方法的值，否则返回一个空的Optional

5.	T get()
    如果在这个Optional中包含这个值，返回值，否则抛出异常：NoSuchElementException

6.	int hashCode()
    返回存在值的哈希码，如果值不存在 返回 0。

7.	void ifPresent(Consumer<? super T> consumer)
    如果值存在则使用该值调用 consumer , 否则不做任何事情。

8.	boolean isPresent()
    如果值存在则方法会返回true，否则返回 false。

9.	<U>Optional<U> map(Function<? super T,? extends U> mapper)
    如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。

10.	static <T> Optional<T> of(T value)
    返回一个指定非null值的Optional。

11.	static <T> Optional<T> ofNullable(T value)
    如果为非空，返回 Optional 描述的指定值，否则返回空的 Optional。

12.	T orElse(T other)
    如果存在该值，返回值， 否则返回 other。

13.	T orElseGet(Supplier<? extends T> other)
    如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果。

14.	<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)
    如果存在该值，返回包含的值，否则抛出由 Supplier 继承的异常

15.	String toString()
    返回一个Optional的非空字符串，用来调试

## 应用示例：
举例一个方法返回值可能为空：
```
public Optional<Map<String, MileageCache>> getMileageFromCache(String deviceId){
        return Optional.ofNullable(MileageCacheUtils.getCacheMileage(deviceId));
    }
```

```
getMileageFromCache(deviceIdStr)
                        .orElse(new HashMap<>()).values().stream()
                        .map(cache -> {
                            MileageRepairData repairData = new MileageRepairData();
                            repairData.setDeviceId(cache.getDeviceId());
                            repairData.setDayTag(cache.getDayTag());
                            repairData.setAppendMileage(cache.getMileage());
                            return repairData;
                        })
                        .forEach(data -> repairPositionData(data, false));
```
尽量不要使用一下形式
```
if（getMileageFromCache.isPresent()） {
}
else {
}
```
    

