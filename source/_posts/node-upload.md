---
title: 使用express搭建文件上传服务器遇到的坑
date: 2017-04-11 13:26:07
tags: [node.js,express]
---

# 说明

最近需要做一个文件上传的功能，但后台还没给API，所以想先用express.js先搭建一个本地文件上传服务器先把功能实现了。实际操作中，发现node.js实现文件上传服务器还有不少坑。所以写篇文章想记下遇到的问题和自己的理解，以备后患。

<!-- more -->

我用到的工具是`formidable@1.1.1`，网上可能更多的推荐是`multer`，但是按教程操作`res.file`或`res.files`始终是undefined，索性改用`formidable`。

还有一种说法express 4.X也已经直接支持文件上传[文章见Cnode](https://cnodejs.org/topic/4f40a4dc0feaaa4424081758)，但是我未能实现，暂时用`formidable`实现，以后有新发现再回头修改。

## formidable github首页

[github/node-formidable](https://github.com/felixge/node-formidable)

## 详细说明

安装`formidable`

```bash
    npm install formidable --save
```

引入`formidable`

``` javascript
    const formidable = require('formidable')
```

初始化`formidable`

``` javascript
    let form = new formidable.IncomingForm();
```

然后用`formidable`实例对象form去解析express的`req`对象

```javascript
    app.all('/upload', (req, res) => {
        form.parse(req, (err, fields, files) => {
            //fields包含上传的文本信息对象
            //files包含上传的文件对象
        });
    }
```

需要注意的是，上传的文件会默认保存在系统临时目录里（可以用`os.tmpdir()`获取），完整文件路径保存在files对象对应文件对象的path参数中。

然后再重命名一下临时文件和删除掉临时文件即可。

```javascript
    Object.keys(files).forEach((v, i) => {
        try{
            fs.writeFileSync('./upload/' + files[v].name, fs.readFileSync(files[v].path))
            fs.unlinkSync(files[v].path)
        }catch(e) {
            console.warn(e)
        }
    })
```

# 客户端的问题

再说说文件上传客户端的问题。阮一峰大神这篇解释挺清楚[文件上传的渐进式增强](http://www.ruanyifeng.com/blog/2012/08/file_upload.html)。所以原理就不再赘述，就在这记录下实现的方法。

## form表单上传

form表单上传是最简单的文件上传,只需要一个form标签，把enctype设为multipart/form-data。action设置为上传路径。缺点是提交后会跳转。基本不会再用。

```html
<form enctype="multipart/form-data" method="POST">
    <input id="file1" type="file" name="file1">
    ...
</form>
```

## 利用iframe的form表单提交

把form表单的action值指向一个同页面隐藏的iframe。此方法页面不会跳转，也不会阻塞页面（传统form上传是同步上传），甚至可以获取到服务器的返回信息。

## ajax上传

ajax上传是现在的主流方法，但是只能兼容IE10以上的高级浏览器。IE8呵呵

```javascript
//一份完整的请求示例
var formData = new FormData();

formData.append('name1', $('#upload').files[0])
formData.append('name2', $('#upload').files[1])
formData.append('name3', $("#text").val())

$.ajax({
    url: url,
    method: "POST",
    data: formData,
    processData: false,     //必须是false, 不然会被转换成字符串。
    contentType: false,     //必须是false
    xhr: function(e) {
        var myXhr = $.ajaxSettings.xhr();
        console.log(myXhr)
        if(myXhr.upload) {
            myXhr.upload.onprogress = function(e) {
                if (e.lengthComputable) {
                    //这个是总进度，多个文件的总进度。
                    progress(e.loaded, e.total, myXhr)
                }
            }
        }
        //一定要return xhr对象，否则不会上传
        return myXhr
    },
    success: function(res){
        console.log(res)
    }
});
```

有几点需要注意的。

文件上传需要用到FormData对象来包装文件，模拟表单提交

```js
var formData = new FormData();
formData.append('name1', $('#upload').files[0])   
formData.append('name2', $('#upload').files[1])
//单个input多文件
formData.append('name3', $("#text").val())
```

用`append`方法给`formData`添加数据。
`dom.files[index]`方法来获取input标签内的文件数据。
console.log(dom.files[index])也许打印出来的内容看起来是个普通的对象，好像并没有包含文件内容。但是事实上这样的确可以把文件上传上去。

用jQuery上传。有两个参数是必须的：
`processData: false`和`contentType: false；`

* processData设置为false。因为data值是FormData对象，不需要对数据做处理。
* contentType设置为false。因为是由<form>表单构造的FormData对象，且已经声明了属性enctype="multipart/form-data"，所以这里设置为false。
* cache设置为false，上传文件不需要缓存。

## ajax进度条

```js
//xhr (默认: 当可用的ActiveXObject（IE）中，否则为XMLHttpRequest)
    // 类型: Function()
    // 回调创建XMLHttpRequest对象。当可用时默认为ActiveXObject（IE）中，否则为XMLHttpRequest。提供覆盖你自己的执行的XMLHttpRequest或增强工厂。
    xhr: function(e) {
        var myXhr = $.ajaxSettings.xhr();
        console.log(myXhr)
        if(myXhr.upload) {
            myXhr.upload.onprogress = function(e) {
                if (e.lengthComputable) {
                    //这个是总进度，多个文件的总进度。
                    progress(e.loaded, e.total, myXhr)
                }
            }
        }
        //一定要return xhr对象，否则不会上传
        return myXhr
    },
    success: function(res){
        console.log(res)
    }
```

## 拖拽方法封装

```js
//初始化拖拽方法。只是要获取文件路径不需要readFile API
function initDrag( selector, callback ) {
    
    let $elem = $(selector);
    $elem.counter = 0;

    $(selector).on('dragenter', function(e) {
        e.preventDefault();
        e.stopPropagation();

        $elem.counter++;
        $elem.addClass( 'highLight' )
    })

    $(selector).on('dragleave', function(e) {
        e.preventDefault();
        e.stopPropagation();

        //修正拖拽方法进入子元素时也会出发dragleave事件的方法
        $elem.counter--;
        if($elem.counter === 0) {
            $elem.removeClass( 'highLight' )           
        }
    })

    $(selector).on('dragover', function(e) {
        e.preventDefault();
        e.stopPropagation();
    })

    $(selector).on('drop', function(e) {
        e.preventDefault();
        e.stopPropagation();

        $elem.counter--;
        if($elem.counter === 0) {
            $elem.removeClass( 'highLight' )
        }
        if($.isFunction(callback)) callback(e);
    })
    
}
```