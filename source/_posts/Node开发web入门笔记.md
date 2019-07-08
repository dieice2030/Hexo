---
title: Node开发web入门笔记（使用Express）
date: 2018-05-19 13:46:11
categories: 伪·技术文章
tags:
---
从开始折腾ASP.NET CORE到现在开始折腾Node.js。虽然已经使用ASP.NET CORE搭建过小型动态网站，但是对于HTTP协议的认识还是基本为零。不得不承认.NET的强大。但我一直认为只有对细节有足够的了解才能走得更远。

# 响应和请求
客户端（浏览器）通过URL或者页面上的交互组件向服务器发出`请求`，服务器经过处理向客户端（浏览器）返回`响应`。

`请求`分为`get`和`post`两种，[具体差别](http://www.w3school.com.cn/tags/att_form_method.asp)。

简单来说`get`快速，但是数据大小有限，不安全；`post`慢，但是数据没有限制，相对安全

在Express中，有专门的对象用以处理不同类型的请求

``` js
var app=express();

app.get('/',response(req,res){  //用以处理get请求

})
app.post('/',response(req,res){  //用以处理post请求

})
```