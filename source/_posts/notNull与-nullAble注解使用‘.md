---
title: '@notNull与@nullAble注解使用‘'
date: 2019-08-06 13:57:14
tags:
---
## @NullAble注解：
1.可以在返回空值的方法上使用
2.可以用于允许为空的变量（字段、局部变量、参数）
3.父类方法使用@NullAble注解则在子类方法中可以具有@NullAble注解或@NotNull注解
4.父类方法参数使用@NullAble注解则需要子类方法参数使用@NullAble注解
## @NotNull注解:
1.应用于返回不为空的方法上
2.应用于不可含空值的变量（字段、局部变量、参数）
3.父类方法批注了@NotNull则子类方法需要批注@NotNull
4.父类方法参数使用@NotNull注解则子类方法参数可以有@NullAble注解或@NotNull注解也可以不写
## 注：
@NullAble注解和@NotNull注解，idea可以帮助我们检测方法的返回值、方法参数以及局部变量是否为空，从而帮护我们减少nullPointerException的发生。
idea配置参考：
<https://blog.csdn.net/u010900754/article/details/91323427>