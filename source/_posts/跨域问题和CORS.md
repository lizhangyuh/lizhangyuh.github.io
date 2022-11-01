---
title: 跨域问题和CORS
category_bar: true
categories: []
date: 2020-10-01 10:00:00
tags: [网络]
---

## 什么是CORS

所谓跨域，通俗来说就是该网站访问了其他origin（源，由域、协议和端口组成）的资源。

浏览器发出的 `XMLHttpRequest` 请求有同源使用限制，默认情况下跨域请求是不被允许的。但是，每个源可以设置哪些其他源可以访问自己的资源，如果一个请求源A在源B的允许请求范围内，那么浏览器就允许请求源A对源B的跨域请求。这种检查机制就是跨源资源共享 (CORS)，是一种基于 HTTP 头的机制。

## 简单请求和非简单请求

在介绍CORS之前，首先要明确两种不同类型的请求，`简单请求` 和 `非简单请求`。 

若请求 满足所有下述条件，则该请求可视为`简单请求`：

使用下列方法之一：

- GET
- HEAD
- POST

请求的Headers只包含以下字段：

- Accept
- Accept-Language
- Content-Language
- Content-Type（需要注意额外的限制）
- 请求中的任意 XMLHttpRequest 对象均没有注册任何事件监听器；XMLHttpRequest 对象可以使用 XMLHttpRequest.upload 属性访问。
- 请求中没有使用 ReadableStream 对象。

Content-Type 的值仅限于下列三者之一：
- text/plain
- multipart/form-data
- application/x-www-form-urlencoded

而不符合上述条件的则为 `非简单请求`。

## CORS预检请求（Preflight request）

浏览器会在必要的时候向服务器发送一个 `OPTIONS` 请求，用来检查服务器是否支持跨域资源共享即CORS。这个请求就叫预检请求（Preflight request）。

请求通常携带 `Access-Control-Request-Method` 和 `Access-Control-Request-Headers`，以及一个 `Origin`  的首部信息。

举一个例子，在实际发送 `DELETE` 请求之前，会先向服务器发起一个预检 `OPTIONS` 请求：

```
OPTIONS /resource/foo
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: origin, x-requested-with
Origin: https://foo.bar.org
```

如果服务器允许，就会相应这个请求，并在response header里面返回允许的请求源和方法：

```
HTTP/1.1 200 OK
Content-Length: 0
Connection: keep-alive
Access-Control-Allow-Origin: https://foo.bar.org
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE
Access-Control-Max-Age: 86400
```

这和开头的两种请求类型有什么关系呢？

简单来说，`简单请求` 不会触发预检请求，而 `非简单请求` 则会触发。这么做的原因也很简单，是否允许CORS是在响应的Header里返回的，而`非简单请求` 都是可能会对服务器数据产生未预期影响的操作，比如删除，修改或者是带有其他Header信息，所以要通过 `OPTIONS` 预检请求来验证请求是否符合CORS规则，同时又不影响服务端数据。

需要注意的是，Response Header里的 `Access-Control-Max-Age` 信息表示在86400秒，也就是24小时内，无需为统一请求再次发起预检请求。当然浏览器也会维护一个最大有效时间，如果Header中的最大有效时间超过了浏览器设置，则不会生效。

## Response Header

跟CORS相关的Response Header，日常工作中注意在服务端，反向代理或者CDN等设置中根据实际情况进行相应设置。

```
// 设置允许访问该资源的外域 URI
Access-Control-Allow-Origin: <origin> | *

// 把允许浏览器访问的头放入白名单
Access-Control-Allow-Methods: <method>[, <method>]*

// 用于预检请求的响应，其指明了实际请求所允许使用的 HTTP 方法
Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header

// 指定了 preflight 请求的结果能够被缓存多久
Access-Control-Max-Age: <delta-seconds>

// 一个布尔值，表示是否允许发送Cookie，如果不需要浏览器传递cookie则不设置该值
Access-Control-Allow-Credentials: true
```

## 参考文献

1. [跨源资源共享（CORS） - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)
2. [跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html)

