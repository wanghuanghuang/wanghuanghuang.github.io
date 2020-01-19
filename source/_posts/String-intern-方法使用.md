---
title: String.intern()方法使用
date: 2019-07-10 14:00:42
tags:
---

开始使用背景：
最近开发时，遇到一个问题:针对同一个String类型的字段，任何用户在第一次点击时可以直接下发指令，成功后在120秒以内不可以多次点击按钮下发指令。之前想直接在下发指令处加一个锁，可是这样的话，会造成任何字段无论是第一次还是重复下发均会排队，这并不是我们想要的结果，这个时候就需要用到String.intern()方法了。

# 一.String的理解
## 1.String不属于java的8种基本数据类型，String是一个对象
基本数据类型：
byte(字节型)、short(短整型)、int(整型)、long(长整型)、float(单精度浮点型)、double(双精度浮点型)、boolean(布尔型)、char(字符型)
对应的包装类：
Byte、Short、Integer、Long、Float、Double、Boolean、Character
基本数据类型存在默认值：
byte((byte)0)、short((short)0)、int(0)、long(0L)、float(0.0F)、double(0.0D)、boolean()、char()

