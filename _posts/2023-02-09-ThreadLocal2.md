---
layout: post
title: ThreadLocal2
date: 2023-02-09 10:47:23
tags: [编程]
toc:  true
---
简单的分析一下 ThreadLocal1.ThreadLocal可以做到数据的非显性赋值，且数据在线程之间是相互隔离的。  

怎么做到的呢，其实原理很简单：给每个线程"绑定"各自的数据(多数情况就是new一个新的对象)。  
简单的讲就是(伪代码)：  
```java
Object obj = new Object()
↓
ThreadLocal.set(obj)
↓
thread.threadLocalMap.add(ThreadLocal,obj)
```
threadLocalMap是Thread的成员变量，ThreadLocalMap.class是ThreadLocal的内部类，ThreadLocal的set(Object)方法就是把这个Object对象保存到ThreadLocalMap中。每个Thread在调用ThreadLocal.set的时候，都会new一个新的Object对象，所以ThreadLocal.get的时候，Thread相当于取到了各自的Object成员变量，因此数据不会相互干扰。

2.为什么不直接给Thread添加成员变量，而要经过ThreadLocal来间接设置呢？我感觉也并不是不行，自定义一个MyThread继承Thread，把想要的数据定义成员变量即可，可以达到相同的目的。但是有个问题，如果Thread1想要的数据类型是Integer，Thread2想要的是String，岂不是又得修改MyThread，假如又出现了Thread3呢...挺麻烦的，还不如直接使用Api定义好的ThreadLocal，针对不用的T使用不同的ThreadLocal<T>  

3.ThreadLocalMap中的EntryEntry才是ThreadLocalMap中真正存储数据的地方，它继承WeakReference，构造函数  
```java
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
```
ThreadLocalMap中的Entry为什么被定义成数组？  
因为同一个Thread可能对应多个ThreadLocal，不同的ThreadLocal.set设置的数据也不一样，这样使用Entry数组来储存所有的数据比较方便。  
Entry的key为什么需要定义成弱引用？  
假设key是强引用，那么：把ThreadLocal引用置为空(=null)的时候，Entry的key依然持有ThreadLocal对象的强引用，那么这个ThreadLocal不会销毁。这种情况下怎么才能销毁对应的Entry对象呢？我认为只能在ThreadLocal置为空的同时，也把对应的Entry置为空，两者同步进行。  
key是弱引用：那么把ThreadLocal引用置为空后，系统gc后，由于对应的Entry的key是弱引用，所以ThreadLocal对象被销毁，此时的Entry应该变成了Entry(null,value)，那value什么时候销毁呢，因为key是null，value已经没有存在的意义。这里，每次使用ThreadLocal.set/get，系统都会调用以下方法销毁key==null的value  
```java
expungeStaleEntry(int staleSlot)
```
既然强引用的情况也能达到销毁Entry的目的，为啥还要设计成弱引用，我觉得设计者的考虑是：  
强引用的情况，每次回收前，需要先调用一个类似ThreadLocal.clear()的函数销毁Entry，再调用ThreadLocal=null。  
弱引用情况，只需要ThreadLocal=null，在gc的时候，系统会自动回收Entry的key对应的ThreadLocal对象。这样，ThreadLocal只需要对外提供get和set接口即可，显得逻辑上更加紧凑。
