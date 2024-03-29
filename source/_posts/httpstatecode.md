---
title: HTTP常用状态码详解
date: 2017-04-04 16:01:51
tags:
 - HTTP
 - 状态码
categories:
 - HTTP
---

HTTP状态码（HTTP Status Code）是用以表示网页服务器HTTP响应状态的3位数字代码。它由 RFC 2616 规范定义的，并得到RFC 2518、RFC 2817、RFC 2295、RFC 2774、RFC 4918等规范扩展。

| 类别   | 说明              |
| ---- | --------------- |
| 1xx  | 请求收到，继续处理       |
| 2xx  | 操作成功收到，分析、接受    |
| 3xx  | 完成此请求必须进一步处理    |
| 4xx  | 请求包含一个错误语法或不能完成 |
| 5xx  | 服务器执行一个完全有效请求失败 |

### 1xx：请求收到，继续处理

| 状态码  | 说明                                       |
| ---- | ---------------------------------------- |
| 100  | 客户端必须继续发送请求                              |
| 101  | 服务器已经理解了客户端的请求，并将通过Upgrade 消息头通知客户端采用不同的协议来完成这个请求 |
| 102  | 处理将被继续执行                                 |

### 2xx：操作成功收到，分析、接受

| 状态码  | 说明                                       |
| ---- | ---------------------------------------- |
| 200  | 请求已成功，请求所希望的响应头或数据体将随此响应返回               |
| 201  | 请求已经被实现，而且有一个新的资源已经依据请求的需要而建立，且其 URI 已经随Location 头信息返回 |
| 202  | 服务器已接受请求，但尚未处理                           |
| 203  | 返回信息不确定或不完整                              |
| 204  | 请求收到，但返回信息为空                             |
| 205  | 服务器完成了请求，用户代理必须复位当前已经浏览过的文件              |
| 206  | 服务器已经完成了部分用户的GET请求                       |
| 207  | 之后的消息体将是一个XML消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码 |

### 3xx：完成此请求必须进一步处理

| 状态码  | 说明                       |
| ---- | ------------------------ |
| 300  | 请求的资源可在多处得到              |
| 301  | 删除请求数据                   |
| 302  | 在其他地址发现了请求数据             |
| 303  | 建议客户访问其他URL或访问方式         |
| 304  | 客户端已经执行了GET，但文件未变化       |
| 305  | 请求的资源必须从服务器指定的地址得到       |
| 306  | 前一版本HTTP中使用的代码，现行版本中不再使用 |
| 307  | 申明请求的资源临时性删除             |

### 4xx：请求包含一个错误语法或不能完成

| 状态码  | 说明                                       |
| ---- | ---------------------------------------- |
| 400  | 错误请求，如语法错误                               |
| 401  | 请求授权失败                                   |
| 402  | 保留有效ChargeTo头响应                          |
| 403  | 请求不允许                                    |
| 404  | 没有发现文件、查询或URl                            |
| 405  | 用户在Request-Line字段定义的方法不允许                |
| 406  | 根据用户发送的Accept拖，请求资源不可访问                  |
| 407  | 类似401，用户必须首先在代理服务器上得到授权                  |
| 408  | 客户端没有在用户指定的饿时间内完成请求                      |
| 409  | 对当前资源状态，请求不能完成                           |
| 410  | 服务器上不再有此资源且无进一步的参考地址                     |
| 411  | 服务器拒绝用户定义的Content-Length属性请求             |
| 412  | 一个或多个请求头字段在当前请求中错误                       |
| 413  | 请求的资源大于服务器允许的大小                          |
| 414  | 请求的资源URL长于服务器允许的长度                       |
| 415  | 请求资源不支持请求项目格式                            |
| 416  | 请求中包含Range请求头字段，在当前请求资源范围内没有range指示值，请求也不包含If-Range请求头字段 |
| 417  | 服务器不满足请求Expect头字段指定的期望值，如果是代理服务器，可能是下一级服务器不能满足请求 |
| 422  | 请求格式正确，但是由于含有语义错误，无法响应                   |
| 423  | 当前资源被锁定                                  |
| 424  | 由于之前的某个请求发生的错误，导致当前请求失败                  |
| 425  | 在WebDav Advanced Collections 草案中定义，但是未出现在《WebDAV 顺序集协议》（RFC 3658）中 |
| 426  | 客户端应当切换到TLS/1.0                          |
| 427  | 请求应当在执行完适当的操作后进行重试                       |

### 5xx：服务器执行一个完全有效请求失败

| 状态   | 说明                                |
| ---- | --------------------------------- |
| 500  | 服务器产生内部错误                         |
| 501  | 服务器不支持请求的函数                       |
| 502  | 服务器作为网关或代理，从上游服务器收到无效响应           |
| 503  | 服务器目前无法使用(由于超载或停机维护)              |
| 504  | 关口过载，服务器使用另一个关口或服务来响应用户，等待时间设定值较长 |
| 505  | 服务器不支持或拒绝支请求头中指定的HTTP版本           |
| 506  | 服务器存在内部配置错误                       |
| 507  | 服务器无法存储完成请求所必须的内容                 |
| 509  | 服务器达到带宽限制                         |
| 510  | 获取资源所需要的策略并没有没满足                  |



Read More:

> [HTTP 返回状态值详解](http://www.cnblogs.com/jinjiangongzuoshi/p/3778883.html)

