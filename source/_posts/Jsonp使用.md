---
title: Jsonp使用
date: 2016-05-03 10:19:19
tags:
---
## 服务端提供数据

假设已经存在两个服务端的接口作为示例，一个返回json数据，一个返回jsonp数据。如下（此处我的接口是用的同一个，根据不同的格式，会返回不同的数据）：

（1）  json接口

> url:  http://localhost:8080/v2/getUserJson.json 

>  返回数据： 

{"code":"","data":"hello jsonp","msg":"获取数据成功","success":true}


（2） jsonp接口

> url: http://localhost:8080/v2/getUserJson.jsonp?callback=handler

> 返回数据：

handler({"code":"","data":"hello jsonp","msg":"获取数据成功","success":true});


## 客户端调用

本地新建一个html文件作为客户端调用测试，可以看一下两者的区别。

```


<html>

<head>

<title>Jsonp</title>

<script src="http://cdn.bootcss.com/jquery/3.0.0-beta1/jquery.js"></script>

<script>



function testCallback(data){

  console.log('test jsonp');

  console.log('get jsonp callback data:'+data.data);

}



$.ajax({

  url:'http://localhost:8080/v2/getUserJson.jsonp?callback=testCallback',

  dataType:'jsonp'

})



  $.ajax({

    url:'http://localhost:8080/v2/getUserJson.jsonp',

    dataType:'jsonp',

    jsonp:'callback',

    jsonpCallback:'handler',

    success:function(data){

      console.log('test ajax jsonp');

      console.log('get ajax success jsonp data name:'+data.data);

    }

  })



$.ajax({

  url:'http://localhost:8080/v2/getUserJson.json',

  dataType:'json',

  success:function(data){

    console.log('test ajax json');

    console.log('final result:'+data);

    console.log('data name:'+data.data);

  }

  })



</script>

</head>

<body></body>

</html>



```

## 结果

![img](/uploads/result.png)

可以看到直接从本地调用json数据的话，会产生一个跨域的问题，而jsonp则不会。

## why？

为什么jsonp不会产生跨域问题，而直接调用json则会产生这么个问题呢？

得先了解下，为什么会产生跨域问题。之所以有这个跨域的问题，就是由于浏览器的同源策略。

> 同源策略，它是由Netscape提出的一个著名的安全策略，现在所有的可支持javascript的浏览器都会使用这个策略。

比如说，浏览器的两个tab页中分别打开了http://www.baidu.com/index.X19Xhtml和http: //www.google.com/index.html，其中，JavaScript1和JavaScript3是属于百度的脚本，而 JavaScript2是属于谷歌的脚本，当浏览器的tab1要运行一个脚本时，便会进行同源检查，只有和www.baidu.com同源的脚本才能被执 行，所谓同源，就是指域名、协议、端口相同。所以，tab1只能执行JavaScript1和JavaScript3脚本，而JavaScript2不能 执行，从而防止其他网页对本网页的非法篡改。

---- 参考自：《[关于javascript跨域以及JSONP的原理与应用](https://segmentfault.com/a/1190000002438126)》



然后，啥是jsonp？

> JSONP 全称是 JSON with Padding ，是基于 JSON 格式的为解决跨域请求资源而产生的解决方案。他实现的基本原理是利用了 HTML 里 <script></script> 元素标签，远程调用 JSON 文件来实现数据传递。

经常写js，以下格式是没有什么问题的：

```

<script src="http://cdn.bootcss.com/jquery/3.0.0-beta1/jquery.js"></script>

```

我们可以引入任何其他网站的一些js文件，jsonp也是利用了这一点。前面的例子是使用ajax封装好的方式去取数据的。最简单的jsonp如下：

```

<script type="text/javascript">
    function jsonpCallback(result) {
        alert(result.msg);
    }
</script>
<script type="text/javascript" src="http://crossdomain.com/jsonServerResponse?jsonp=jsonpCallback"></script>
```

只要该接口返回如下格式的数据，此处回调方法即可拿到数据：

```

jsonpCallback({ msg:'this  is  json  data'})
```

