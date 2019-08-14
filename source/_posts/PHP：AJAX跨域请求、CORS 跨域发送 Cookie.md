---
title: AJAX跨域请求、CORS 跨域发送 Cookie
abbrlink: 6d5007a7
date: 2017-05-15 16:06:48
tags:
  - PHP
  - 后端
categories:
  - 技术
  - PHP
---
在 Web 页面中可以随意地载入跨域的图片、视频、样式等资源， 但 AJAX 请求通常会被浏览器应用同源安全策略，禁止获取跨域数据，以及限制发送跨域请求。 虽然有多种方法利用资源标签进行跨域，但能够进行的数据交互非常有限。 在 2014 年 W3C 发布了 CORS Recommendation 来允许更方便的跨域资源共享。 默认情况下浏览器对跨域请求不会携带 Cookie，但鉴于 Cookie 在身份验证等方面的重要性， CORS 推荐使用额外的响应头字段来允许跨域发送 Cookie。
客户端代码
在open XMLHttpRequest后，设置withCredentials为true即可让该跨域请求携带 Cookie。 注意携带的是目标页面所在域的 Cookie。
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.withCredentials = true;
xhr.send();

如果你在使用 jQuery，可以通过xhrFields来设置：
$.ajax({
   url: a_cross_domain_url,
   xhrFields: {
      withCredentials: true
   }
});

如果你在使用 Zepto，抱歉没有办法。因为 Zepto 会在open之前设置withCredentials。 根据WHATWG 的 XHR 标准在open之后设置是不合法的， 虽然多数浏览器不抛出错误但仍然不会携带 Cookie。
True when user credentials are to be included in a cross-origin request. False when they are to be excluded in a cross-origin request and when cookies are to be ignored in its response. Initially false. When set: throws an InvalidStateError exception if state is not unsent or opened, or if the send() flag is set. – WHATWG XMLHttpRequest
Access-Control-Allow-Credentials
只设置客户端当然是没用的，还需要目标服务器接受你跨域发送的 Cookie。 否则会被浏览器的同源策略挡住：

服务器同时设置Access-Control-Allow-Credentials响应头为"true"， 即可允许跨域请求携带 Cookie。
Access-Control-Allow-Origin
除了Access-Control-Allow-Credentials之外，跨域发送 Cookie 还要求 Access-Control-Allow-Origin不允许使用通配符。 事实上不仅不允许通配符，而且只能指定单一域名：
If the credentials flag is true and the response includes zero or more than one Access-Control-Allow-Credentials header values return fail and terminate this algorithm. –W3C Cross-Origin Resource Sharing
否则，浏览器还是会挡住跨域请求：

计算 Access-Control-Allow-Origin
既然Access-Control-Allow-Origin只允许单一域名， 服务器可能需要维护一个接受 Cookie 的 Origin 列表， 验证 Origin 请求头字段后直接将其设置为Access-Control-Allow-Origin的值。 （这一实践来自 Stackoverflow） 值得注意的是在 CORS 请求被重定向后 Origin 头字段会被置为 null。 此时可以选择从Referer头字段计算得到Origin。
在正确配置的情况下，在 Chrome Network 就可以看到 Cookie 请求头被跨域发送了 （注意 Host 和Referer不同域，但仍然带了Cookie）：
Accept:*/*
Accept-Encoding:gzip, deflate, sdch, br
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6,nl;q=0.4,zh-TW;q=0.2,fr;q=0.2,de;q=0.2,ja;q=0.2
Connection:keep-alive
Cookie:auhtor:harttle; _gat=1; _ga=GA1.1.221305049.1482947002
Host:dest.com:4001
Origin:http://index.com:4001
Referer:http://index.com:4001/
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36

服务器端代码
const express = require('express');

var app = express();
app.get('/specific-allow-origin-with-credentials', (req, res) => {
    res.set({
        'Access-Control-Allow-Origin': 'http://index.com:4001',
        'Access-Control-Allow-Credentials': true
    });
    res.status(200).end('I got your cookie: ' + req.headers.cookie);
});
app.listen(4001, () => console.log('listening to 4001'));