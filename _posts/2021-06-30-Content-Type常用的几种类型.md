---
layout: post
title: Content-Type常用的几种类型
date: 2021-06-30 17:46:55
tags: [编程]
toc:  true
---

Content-Type: application/json ：  
请求体中的数据会以json字符串的形式发送到后端  

Content-Type: application/x-www-form-urlencoded：  
请求体中的数据会以普通表单形式（键值对）发送到后端  

Content-Type: multipart/form-data：  
它会将请求体的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，  
也可以上传文件。  

举个例子：  
Content-Type: application/x-www-form-urlencoded  
在后端使用springmvc框架写api接口，若使用@RequestParam注解，接收的参数是来自HTTP请求体或请求url的QueryString，前端那边主要使用键值对的形式传参，  
GET请求，后端可以直接读取到传参  
POST请求，后端无法获取到传参，因为POST请求的时候，参数是放在请求体之中，通过@RequestParam注解是无法获取到传参的。  
解决方案1：后端改使用@ResponseBody，以实体类的形式是可以接收得到。  
解决方案2：后端不改，前端这边使用QS库，qs.stringify()可以将对象序列化成URL的形式，以&进行拼接进行传输。那么@RequestParam是可以正常解析到传参的。  
保持上述案例不变，  
Content-Type: application/json  
如果保持以键值对的方式传参请求上述接口，会报400“某个必须参数没有出现”的错误，因为application/json类型的请求体是将参数放在json字符串进行传递的，需要将后端的请求参数进行改变成String类型才能进行正常解析。  
