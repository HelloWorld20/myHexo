---
title: jsonp跨域，并谈谈其他跨域的问题
date: 2017-06-20 08:00:00
tags: [jsonp,跨域]
---

# 说明

公司小伙伴在用jQuery做jsonp跨域请求时遇到了问题，需要我的帮忙。然而虽然了解jsonp的原理，但是以前从来动手实现过，遇到问题还是一头雾水，最终啥都没帮上，还浪费了时间，唉。回家后自己实现了一下。。。。

<!-- more -->

这篇[简书](http://www.jianshu.com/p/38449d9452a7)写了n个demo，跨域分类介绍得很全。

## jsonp原理

1. 浏览器考虑到安全性，禁止了不同源的请求。
2. 但是html的`<script>`标签例外。

## jsonp

``` javascript
//jQuery

$.ajax({
    url: 'xxx',
    dataType: 'jsonp',
    method: 'get',
    jsonp: "callbackparam",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(默认为:callback)
    jsonpCallback:"success_jsonpCallback",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名
    success: function(){}
    })


```

然后发出的请求大概是这样的一个get请求。

```
http://XXX?callback=jQuery112106434915643629524_1497952489640&_=1497952489641
```

原生的话应该是这样的写的。

``` html
<script>
function callback(res) {
    document.write(res)
}
</script>
<script src="http://xxx.com/xxx?jsonpCallback=callback"></script>
```

其实就等价于

``` html

<script>   
function callback(res) {
    document.write(res)
}
</script>
<script>callback('server data')</script>

```

当然，jsonp也需要后台的支持。返回一段约定的JavaScript方法调用

``` javascript
app.get('/', function (req, res) {
    var callbackName = req.query.callback;   // myFunction
    res.send(callbackName+"({'message': 'hello world from JSONP!🙃'});");
    // myFunction({'message': 'hello world from JSONP!'})
    // 一个带参数的执行函数
})
```

远程返回的一段javascript执行了本地定义好的方法，并把需要的参数传给该方法即可达到获取服务器数据的效果

## 需要注意的几点

* `<script>`标签请求js是`GET`请求，并不支持其他类型的请求。所以用`POST`方法做`jsonp`请求是错的（当然，jQuery已经主动帮你写成`GET`了

# 关于jQuery奇怪的报错

当用jQuery做jsonp请求是，可能会报`Uncaught SyntaxError: Unexpected token :`错。常理会认为某个地方出现了语法错误。其实不然。错误的地方在于`接口不支持`。返回的结果jQuery无法处理。才会报这个错。语法错误并不是返回的数据语法错误。

# CORS跨域

顺便介绍另一个常用的跨域解决办法--CORS跨域

原理是服务器在http请求头里写上`Access-Control-Allow-Origin`。声明那些域名可以接收该请求即可。

比如说

``` javascript
app.get('/', (req, res) => {
    res.set('Access-Control-Allow-Origin', 'http://localhost:3000'); // 设置允许跨域的origin，允许3000端口访问本端口（3001）
    res.send("Hello world from CROS.😡");   // 空格部分为表情，可能在编辑器不会显示
});
```

