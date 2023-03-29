---
layout: post
title: Lifecycle原理
date: 2023-03-29 14:25:13
tags: [编程]
toc:  true
---
**一.先下结论**  
Lifecycle实现的根本原理：即在activity的onCreate，onStart..等生命周期函数中回调Lifecycle相对应的回调函数。  

**二.分析LifecycleRegistry**  
对Activity生命周期的监测，都是从lifecycle.addObserver(LifecycleObserver)开始，而addObserver()的具体实现类是LifecycleRegistry，
LifecycleRegistry是Lifecycle的实现类  
1.LifecycleRegistry扮演的是被观察者的角色(主题)，而被添加进来的LifecycleObserver是观察者，当Activity的生命周期发生改变的时候，
LifecycleRegistry会告知每个订阅者(下文将这种生命周期的告知简称为"通知")：  
**LifecycleRegistry -> LifecycleObserver**  
2.LifecycleRegistry为什么知道Activity的生命周期的变化的？  
因为这些Activity都维护有一个LifecycleRegistry变量，但是并不是所有的Activity都维护有lifecycle，从基类ComponentActivity开始，才实现了
```java
    @NonNull
    Lifecycle getLifecycle();
```
提一下，虽然FragmentActivity是ComponentActivity的子类，但是FragmentActivity对getLifecycle()进行了重写。  
因此当Activity的生命周期变化时，发生通知：  
**Activity -> LifecycleRegistry -> LifecycleObserver**  

**三.分析ReportFragment**  
1.SDK<29(即Android10)的情况  
ComponentActivity在onCreate函数中会注入一个ReportFragment，ReportFragment是没有布局的  
```java
 manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
```
Activity和ReportFragment之间存在"绑定"的关系，例如当Activity的onCreate函数回调的时候，ReportFragment的onCreate同样进行回调，那么就可以依托
ReportFragment的生命周期反馈出宿主Activity的情况  
2.SDK>=29的情况
这时候，其实已经和ReportFragment没多大关系了，解除了通过Fragment绑定宿主Activity体现生命周期的办法，而直接使用：
```java
  static class LifecycleCallbacks implements Application.ActivityLifecycleCallbacks {
  
          static void registerIn(Activity activity) {
              activity.registerActivityLifecycleCallbacks(new LifecycleCallbacks());
          }
}
```
LifecycleCallbacks是ReportFragment的静态内部类，通过注册Application.ActivityLifecycleCallbacks实现Activity生命周期的监听，
能监听就能广播给每位订阅者。  
3.那么进一步完善这个通知过程：
**Activity -> ReportFragment -> LifecycleRegistry -> LifecycleObserver**  

**四.分析ActivityLifecycleCallbacks**  
1.Application的ActivityLifecycleCallbacks为啥能够感知Activity的生命周期？
Activity维护了一个存放ActivityLifecycleCallbacks(下面简称callbacks)的ArrayList，当Activity生命周期函数回调时，同时通知给ArrayList中的
callbacks,如下：
```java
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        ...
        dispatchActivityCreated(savedInstanceState);
    }  

   protected void onStart() {
        ...
        dispatchActivityStarted();
    }
```

**五.抛开源码细节部分，我认为Lifecycle的实现和回调过程是：**  
**Activity(被观察者) -> ReportFragment(中间件) -> LifecycleRegistry(分发) -> LifecycleObserver(观察者)**  

