---
layout: post
title: Lifecycle原理
date: 2023-03-29 14:25:13
tags: [编程]
toc:  true
---
**一.先下结论**  
Lifecycle实现的根本原理：即在activity的onCreate，onStart..等生命周期函数中回调Lifecycle相对应的回调函数。  

