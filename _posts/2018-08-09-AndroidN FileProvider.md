---
layout: post
title: AndroidN FileProvider
date: 2018-08-09 10:14:32
tags: [编程]
toc:  true
---
FileProvider使用方法  

paths 类型:  
```
<files-path name=\"name\" path=\"path\" />
返回 Context.getFilesDir() + path

<cache-path name=\"name\" path=\"path\" />
返回 Context. getCacheDir() + path

<external-path name=\"name\" path=\"path\" />
返回 Environment.getExternalStorageDirectory() + path

<external-files-path name=\"name\" path=\"path\" />
返回 Context.getExternalFilesDir(null) + path

<external-cache-path name=\"name\" path=\"path\" />
返回 Context.getExternalCacheDir() + path

<external-media-path name=\"name\" path=\"path\" />
返回 Context.getExternalMediaDirs() + path

<root-path name=\"zixie_file_provider\" path=\"\" />
```

返回整个存储目录  

name属性：  
指明了 FileProvider 在content uri中需要添加的部分  
path属性：  
对应的路径的子路径  
path值为点符号（"."）时，该根目录下所有的文件夹都可以临时授权访问  

举个例子  
```
<external-path
name=\"my_images\"
path=\"Pictures\" />
```
返回Environment.getExternalStorageDirectory() + Pictures储存路径  
假设包名为com.test.app  
当访问文件 content://com.test.app/my_images/123.jpg 时，就会找到path路径Environment.getExternalStorageDirectory() + “/Pictures/” 并查找 123.jpg 图片