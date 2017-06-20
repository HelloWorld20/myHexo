---
title: jsonp跨域，并谈谈其他跨域的问题
date: 2017-06-20 08:00:00
tags: [jsonp,跨域]
---

# 说明

公司小伙伴在用jQuery做jsonp跨域请求时遇到了问题，需要我的帮忙。然而虽然了解jsonp的原理，但是以前动手实现过，遇到问题还是一头雾水，最终啥都没帮上，还浪费了时间，唉。回家后自己实现了一下。。。。

<!-- more -->

这篇[简书](http://www.jianshu.com/p/38449d9452a7)写了n个demo，跨域分类介绍得很全。

## jsonp原理

## jsonp

``` javascript
//jQuery

$.ajax({
    url: 'xxx',
    dataType: 'jsonp',
    jsonp: "callbackparam",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(默认为:callback)
    jsonpCallback:"success_jsonpCallback",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名
    success: function(){}
    })


```

然后发出的请求大概是这样的一个get请求。

```
http://XXX?callback=jQuery112106434915643629524_1497952489640&_=1497952489641
```


``` javascript
<script>

</script>
```