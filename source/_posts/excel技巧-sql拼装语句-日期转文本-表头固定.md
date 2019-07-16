---
title: excel技巧-sql拼装语句+日期转文本+表头固定
date: 2019-07-09 17:36:32
tags:
---

# 一.excel表头固定

选择想要固定行的下一行的第一个单元格（例如：固定第1、2、3行，就选择第四行的第一个单元格），选择 视图-> 冻结窗格
  注：冻结列 同样适用

# 二.日期转文本
使用公式：=TEXT(要转换的日期单元格,"yyyy-mm-dd hh:mm:ss")

# 三.excel 拼接sql
="update dd_new.veh_device set INSTALL_DATE='"&F2&"',INSTALL_LOCATION='"&G2&"',INSTALL_ADDRESS='"&H2&"',INSTALL_TYPE='"&D2&"',INSTALLER='"&C2&"' where id='"&A2&"'"