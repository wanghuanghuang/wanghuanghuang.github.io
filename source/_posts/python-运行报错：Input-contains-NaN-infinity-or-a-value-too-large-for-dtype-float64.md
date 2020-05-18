---
title: >-
  python 运行报错：Input contains NaN, infinity or a value too large for
  dtype('float64')
date: 2020-05-15 15:38:19
tags:
---

背景：产品急需车辆的推算家庭地址，但这个是Python程序写的，我并不熟悉，所以搭建环境，运行程序时出现：
```
ERROR:root:Input contains NaN, infinity or a value too large for dtype('float64').
Traceback (most recent call last):
  File "D:\idea_project\Python_address_home_work\vehicle_stop.py", line 127, in dbscan_cluster
    db = DBSCAN(eps=epsilon, min_samples=2, algorithm='ball_tree', metric='haversine', n_jobs=-1).fit(np.radians(coords))
  File "D:\Downloads\Python\lib\site-packages\sklearn\cluster\dbscan_.py", line 320, in fit
    X = check_array(X, accept_sparse='csr')
  File "D:\Downloads\Python\lib\site-packages\sklearn\utils\validation.py", line 573, in check_array
    allow_nan=force_all_finite == 'allow-nan')
  File "D:\Downloads\Python\lib\site-packages\sklearn\utils\validation.py", line 56, in _assert_all_finite
    raise ValueError(msg_err.format(type_err, X.dtype))
```

调查过程：
1.在log日志处打上断点并将需要观察的变量值加入到监控中
2.根据控制台打印的错误日志，报错的三个地方均打上断点
3.运行程序

发现是在运行' db = DBSCAN(eps=epsilon, min_samples=2, algorithm='ball_tree', metric='haversine', n_jobs=-1).fit(np.radians(coords))'
这行代码的dbscan.fit(self, X, y=None, sample_weight=None)中的校验传值数组的 'X = check_array(X, accept_sparse='csr')'的校验数组是否超限时报错'if force_all_finite:allow_nan=force_all_finite == 'allow-nan')'。
这里附上'force_all_finite'代码：
```python
def _assert_all_finite(X, allow_nan=False):
    """Like assert_all_finite, but only for ndarray."""
    if _get_config()['assume_finite']:
        return
    X = np.asanyarray(X)
    # First try an O(n) time, O(1) space solution for the common case that
    # everything is finite; fall back to O(n) space np.isfinite to prevent
    # false positives from overflow in sum method.
    is_float = X.dtype.kind in 'fc'  ## 校验数组值得类型 这里是float 结果为true
    if is_float and np.isfinite(X.sum()):  ## 校验数组值相加是否超限了
        pass
    elif is_float:  
        msg_err = "Input contains {} or a value too large for {!r}."
        if (allow_nan and np.isinf(X).any() or
                not allow_nan and not np.isfinite(X).all()):
            type_err = 'infinity' if allow_nan else 'NaN, infinity'
            raise ValueError(msg_err.format(type_err, X.dtype))
```
看代码可以得出结论 我们报错是因为在判断数组值相加是否超限np.isfinite(X.sum()) 不通过 走到else分支中

解决方案是 限制数组的值相加后的和 或者限制数组的长度




