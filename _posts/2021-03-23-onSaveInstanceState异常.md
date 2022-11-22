---
layout: post
title: onSaveInstanceState异常
date: 2021-03-23 18:06:05
tags: [编程]
toc:  true
---
**出现bug**   
"java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState"  

在使用DialogFragment的时候,出现上面这个异常，具体场景：
A页面跳转到B页面，在B页面中发送EventBus后finish掉本页面，在A页面的EventBus监听函数中调用DialogFragment.show()方法，出现上述异常且crash。  

**分析原因**   
以下源码基于androidx。  
字面意思：“不能在onSaveInstanceState之后执行该操作”，什么操作呢？  
简单看下在show的过程中执行了哪些操作(中间省略掉一下无关的代码)：  
```java
public void show(@NonNull FragmentManager manager, @Nullable String tag) {
mDismissed = false;
mShownByMe = true;
FragmentTransaction ft = manager.beginTransaction();
ft.add(this, tag);
ft.commit();
}
```
commit最终执行了BackStackRecord中的：  
```java
int commitInternal(boolean allowStateLoss) {
...
mManager.enqueueAction(this, allowStateLoss);
}
```
继续enqueueAction中：  
```java
public void enqueueAction(OpGenerator action, boolean allowStateLoss) {
 if (!allowStateLoss) {
  checkStateLoss();
  }
}
```
看下checkStateLoss()：  
```java
private void checkStateLoss() {
if (isStateSaved()) {
throw new IllegalStateException(
"Can not perform this action after onSaveInstanceState");
  }
}
@Override
public boolean isStateSaved() {
// See saveAllState() for the explanation of this. We do this for
// all platform versions, to keep our behavior more consistent between
// them.
return mStateSaved || mStopped;
}
```

终于看到抛出这个异常的地方。  
当allowStateLoss为false的时候，mStateSaved或mStopped为true，会抛出这个异常。  
看下这两个方法：  
```java
@Override
public int commit() {
return commitInternal(false);
}

@Override
public int commitAllowingStateLoss() {
return commitInternal(true);
}
```
唯一区别，就是传参allowStateLoss的差别，commit()方法传的false。  
剩下的问题，什么时候mStateSaved为true呢，继续追踪源码，我来简单的捋一下整个过程。  
```java
FragmentActivity=>onSaveInstanceState()
|
mFragments=>saveAllState()
|
FragmentController=>saveAllState()
|
FragmentManagerImpl=>saveAllState()
|
mStateSaved = true;
```
也就是FragmentActivity在调用onSaveInstanceState的时候，同时会让mFragments通过saveAllState()方法保存自我的状态，saveAllState方法中会调用Fragment的onSaveInstanceState()方法(onSaveInstanceState是一个空函数供子类重写保存一些自定义的数据)，也正是此时mStateSaved被置为true。  
现在逻辑上通了，针对于一个Fragment，当发生页面跳转的时候，会触发onPause->onStop->onSaveInstanceState，当onSaveInstanceState()函数已被触发的时候，再去commit()就会抛出异常。  
所以，上面这个bug产生的原因就是：  
当Eventbus发送事件的时候，由于Eventbus的机制，直接通过反射+注解调用到B页面注册的函数，这个过程是瞬时的，肯定是在A页面还未恢复至onStart状态的情况下，就立刻执行了ft.commit()方法，直接导致触发了上面的bug。  

那么，是不是在A页面的状态已经发生改变的情况下，再调用ft.commit()方法，就不会出现上面的bug了嗯？答案是对的。因为：  
```java
public void dispatchStart() {
mStateSaved = false;
mStopped = false;
dispatchStateChange(Fragment.STARTED);
}
```

当生命周期处于发生改变比如在onStart()的时候，mStateSaved和mStopped都为false。  

**重现场景** 
了解了问题所在原因后，能出现以上bug的常见情况，应该有：  
1.发送EventBus类似的事件  
2.发送广播？应该也是类似的情形  
3.异步操作，回调函数中执行了改变commit操作，但是宿主的Fragment状态已保存，比如页面跳转或者home键等。  
**解决方案**  
针对上面这个bug可以用以下几种解决方案：  
1.使用commitAllowingStateLoss方式进行显示DialogFragment，但是DialogFragment并不提供commitAllowingStateLoss的show()方法，因为DialogFragment的出现好像就是为了弥补普通Dialog无法方便保存状态的痛点，所以使用commitAllowingStateLoss进行提交岂不是违背了初衷。因此，想使用commitAllowingStateLoss只能通过反射，来重写DialogFragment的show()方法即可。  
2.使用onActivityResult，在该回调函数中进行显示DialogFragment，因为宿主FragmentonActivity 的onActivityResult函数中会先调用mFragments.noteStateNotSaved()将 mStateSaved和mStopped置为false(即状态处于未保存状态)，再调用 targetFragment.onActivityResult。
