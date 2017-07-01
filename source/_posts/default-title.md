---
title: 关于toString、valueOf自动执行的问题
date: 2017-04-07 16:33:07
tags: [前端技术]
---

原文：[JavaScript 对象转换之toString、valueOf](http://frontenddev.org/link/convert-the-tostring-the-valueof-javascript-object.html)

首先，JavaScript中有3个基本类型，String、Number和Boolean。JavaScript在某些情况下会将数据转换成基本类型，及JavaScript会自动调用toString和valueOf方法来完成数据转换。

<!-- more -->

### 转换成String

1. 如果toString方法存在并且返回“原始类型”，返回toString的结果。
2. 如果toString方法不存在或者返回的不是“原始类型”，调用valueOf方法，如果valueOf方法存在，并且返回“原始类型”数据，返回valueOf的结果。
3. 其他情况，抛出错误。

### 转换成Number

1. 如果valueOf存在，且返回“原始类型”数据，返回valueOf的结果
2. 如果toString存在，且返回“原始类型”数据，返回toString的结果。
3. 报错。

### 转换成Boolean

按照一定规则转换成Boolean

|值             | 布尔值                     | 
|:------------- |---------------------------:|
|true/false     | true.false                 |
|undefined,null | false                      |
|Number         |0,NaN => false;其他=> true; |
|String         |"" => false; 其他 => true;  |
|Object         | true                       |    

