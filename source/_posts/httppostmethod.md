---
title: HTTP POST提交数据分常见方式
date: 2018-05-13 14:17:00
tags:
 - HTTP
 - POST
categories:
 - HTTP
---

[HTTP/1.1协议](http://www.ietf.org/rfc/rfc2616.txt) 规定的 HTTP 请求方法有 OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE、CONNECT这几种。其中 POST 一般用来向服务端提交数据，我们在写类似接口文档这种被业务方调用的服务时，涉及到 POST 的默认提交方式基本都是：`application/x-www-form-urlencoded` 。本 文主要讨论 POST 提交数据的几种方式。

我们知道，HTTP 协议是以 **ASCII** 码传输，建立在 TCP/IP 协议之上的应用层规范。规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体。类似于下面这样：

> <method> <request-URL> <version>
>
> <headers>
>
> <entity-body>

协议规定 POST 提交的数据必须放在消息主体（`entity-body`）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。

但是，数据发送出去，还要服务端解析成功才有意义。一般服务端语言如 java php python 等，以及它们的 framework，都内置了自动解析常见数据格式的功能。服务端通常是根据请求头（headers）中的 `Content-Type `字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说到 POST 提交数据方案，包含了 `Content-Type` 和消息主体编码方式两部分。下面进入正题。

### application/x-www-form-urlencoded

这种方式是最常见的 POST 提交数据的方式。浏览器的原生 `<form>` 表单，如果不设置 `enctype` 属性，那么最终就会以 `application/x-www-form-urlencoded` 方式提交数据。请求类似于下面这样（无关的请求头在本文中都省略掉了）：

```
POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8
title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```

首先，Content-Type 被指定为 `application/x-www-form-urlencoded` ；其次，提交的数据按照 `key1=val1&key2=val2` 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。

通常，我们用 Ajax 提交数据时，也是使用这种方式。例如 [JQuery](http://jquery.com/) 和 [QWrap](http://www.qwrap.com/) 的 Ajax，`Content-Type` 默认值都是 `application/x-www-form-urlencoded;charset=utf-8` 。

### multipart/form-data

在使用表单上传文件时，必须让 `<form>` 表单的 `enctype` 等于 `multipart/form-data`。直接来看一个请求示例：

```
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

这个例子稍微复杂点。首先生成了一个 `boundary` 用于分割不同的字段，为了避免与正文内容重复，`boundary` 很长很复杂。然后 `Content-Type ` 里指明了数据是以 `multipart/form-data` 来编码，本次请求的 `boundary` 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 `--boundary` 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 `--boundary--` 标示结束。关于 ` multipart/form-data ` 的详细定义，请前往 [rfc1867](http://www.ietf.org/rfc/rfc1867.txt) 查看。这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

以上提到的两种 POST 数据的方式，都是浏览器原生支持的，而且现阶段标准中原生 \<form\> 表单也 [只支持这两种方式](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4)（通过 `<form>` 元素的 `enctype` 属性指定，默认为 `application/x-www-form-urlencoded`。其实 `enctype` 还支持 `text/plain`，不过用得非常少）。

随着越来越多的 Web 站点，尤其是 WebApp，全部使用 Ajax 进行数据交互之后，我们完全可以定义新的数据提交方式，给开发带来更多便利。

### application/json

`application/json ` 这个 `Content-Type` 作为响应头大家肯定不陌生。实际上，现在越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。JSON 格式支持比键值对复杂得多的结构化数据，这一点也很有用。

Google 的 [AngularJS](http://angularjs.org/) 中的 Ajax 功能，默认就是提交 JSON 字符串。例如下面这段代码：

```
var data = {'title':'test', 'sub' : [1,2,3]};
$http.post(url, data).success(function(result) {
    ...
});
```

最终发送的请求是：

```
POST http://www.example.com HTTP/1.1 
Content-Type: application/json;charset=utf-8

{"title":"test","sub":[1,2,3]}
```

这种方案，可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口。各大抓包工具如 Chrome 自带的开发者工具、Firebug、Fiddler，都会以树形结构展示 JSON 数据，非常友好。但也有些服务端语言还没有支持这种方式，例如 php 就无法通过 $\_POST 对象从上面的请求中获得内容。这时候，需要自己动手处理下：在请求头中 `Content-Type` 为 `application/json` 时，从 `php://input` 里获得原始输入流，再 `json_decode` 成对象。一些 php 框架已经开始这么做了。

当然 AngularJS 也可以配置为使用 `x-www-form-urlencoded ` 方式提交数据。如有需要，可以参考 [这里](http://victorblog.com/2012/12/20/make-angularjs-http-service-behave-like-jquery-ajax/)。

### text/xml

这种方式目前几乎很少在用，因为 JSON 方式更加灵活方便，而 xml 方式显得比较臃肿和繁琐。`XML-RPC（XML Remote Procedure Call）` 协议是一套允许运行在不同操作系统、不同环境的程序实现基于 Internet 过程调用的规范和一系列的实现。这种远程过程调用使用 HTTP 作为传输协议，XML 作为传送信息的编码格式。

典型的 `XML-RPC` 请求是这样的：

```
POST http://www.example.com HTTP/1.1 
Content-Type: text/xml

<?xml version="1.0"?>
<methodCall>
    <methodName>examples.getStateName</methodName>
    <params>
        <param>
            <value><i4>41</i4></value>
        </param>
    </params>
</methodCall>
```