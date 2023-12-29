---
layout: post
title: ViewModel原理
date: 2023-03-30 10:22:23 +0800
tags: [编程]
toc:  true
---

### 先下结论
ViewModel的作用，为UI层提供数据，并且能较长时间保存数据直到Activity最终Finished，什么叫最终Finished呢，我的理解是Activity被内存彻底回收。  

**1.为什么ViewModel可以做到长时间保存数据？**  
因为ViewModel是Activity的成员变量，理论上只要不主动置为null，它会跟随Activity的存在而一直存在。  

**2.Activity是怎样储存ViewModel的？**  
基类ComponentActivity存有一个ViewModelStore对象，ViewModelStore很简单，拥有一个HashMap，专门用来处理所有的ViewModel，它们之间的关系：  
**Activity -> ViewModelStore -> ViewModel**  

**3.ViewModel的自动回收**  
当onDestroy()回调的时候，用isChangingConfigurations()用来判断是否为配置变化导致的重启，如果不是，则认为Activity被正常的回收，所有的ViewModel的也随同clear，这也解释了为啥ViewModel能在屏幕旋转后依然能够保存数据。  
```java
        getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source,
                    @NonNull Lifecycle.Event event) {
                if (event == Lifecycle.Event.ON_DESTROY) {
                    // Clear out the available context
                    mContextAwareHelper.clearAvailableContext();
                    // And clear the ViewModelStore
                    if (!isChangingConfigurations()) {
                        getViewModelStore().clear();
                    }
                }
            }
        });
```
