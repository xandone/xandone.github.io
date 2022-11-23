---
layout: post
title: 从setContentView()说起
date: 2019-07-01 20:04:05
tags: [编程]
toc:  true
---
基本上任何一个Activity的布局设置都是从setContentView开始的。(以下源码，均参照android-27版本)  

那么：  

**一、setContentView做了哪些操作？**  
1.会委托给AppCompatDelegate，调用其setContentView方法  
```java
@Override
public void setContentView(View view, ViewGroup.LayoutParams params) {
getDelegate().setContentView(view, params);
}
```
AppCompatDelegate是个抽象类，初始化的时候会对当前SDK版本进行判断，但是最终还是调用的AppCompatDelegateImplV9(实现类)中setContentView  
```java
@Override
public void setContentView(int resId) {
ensureSubDecor();
ViewGroup contentParent = (ViewGroup) mSubDecor.findViewById(android.R.id.content);
contentParent.removeAllViews();
LayoutInflater.from(mContext).inflate(resId, contentParent);
mOriginalWindowCallback.onContentChanged();
}
```
2.AppCompatDelegate的作用主要是通过createSubDecor()加载根布局，根据源码的执行流程，大致分以下几个步骤  
步骤1，加载当前Activiy的主题属性，并进行相关的设置，比如常见的requestWindowFeature设置。  
步骤2，执行函数mWindow.getDecorView()，主要作用是初始化phoneWindow中的DecorView布局  
步骤3，执行函数mWindow.setContentView(subDecor)，围观一下源码，找到phoneWindow中的setContentView()方法，关键代码
mContentParent.addView(view, params)，将AppCompatDelegate中的根布局作为子布局添加到phoneWindow的DecorView中，可见，
phoneWindow的DecorView才是Activity的真正跟布局  
步骤4，最后才将activity设置的view添加到AppCompatDelegate的mSubDecor布局之中。  
3.通过分析activity的布局view的加载过程可见，activity本身并不直接执行布局的具体加载，而是委托window类管理view的加载  

**二、怎样添加一个window？**  
一个简单的demo  
```java
WindowManager windowManager = getWindowManager();
WindowManager.LayoutParams params = new WindowManager.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
ViewGroup.LayoutParams.WRAP_CONTENT);
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
params.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
} else {
params.type = WindowManager.LayoutParams.TYPE_PHONE;
}
params.width = 200;
params.height = 200;
params.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;
params.token = getWindow().getDecorView().getWindowToken();
params.gravity = Gravity.LEFT | Gravity.TOP;
ImageView view = new ImageView(WindowActivity.this);
view.setBackgroundResource(R.color.colorPrimary);
windowManager.addView(view, params);
```
在activity中新建一个系统级别的window，这话听起来有点拗口，但是我觉得可以这么理解，  
首先看看windowManager.addView(view, params);过程中做了些什么，Activity中：  
mWindowManager = mWindow.getWindowManager();可见，activity中的windowManager使用的还是window中的windowManager，然而WindowManager是个接口，具体的实现是WindowManagerImpl类，然后委派给WindowManagerGlobal进行addView操作,通过查看源码可以发现，WindowManagerGlobal是单例模式向外提供。addView加载过程部分源码  
```java
private final ArrayList<View> mViews = new ArrayList<View>();
private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
private final ArrayList<WindowManager.LayoutParams> mParams = new ArrayList<WindowManager.LayoutParams>();
root = new ViewRootImpl(view.getContext(), display);
view.setLayoutParams(wparams);
mViews.add(view);
mRoots.add(root);
mParams.add(wparams);
```
每个布局都对应一个独自的ViewRootImpl和params。mViews，mRoots，mParams分别储存。ViewRootImpl是所有View渲染的关键类，同时包含了xml的解析器方法，这里就不详细分析了，可以单独写篇文章。  

**三、window是怎样和activity联系到一起的？**  
Activity在启动过程中，通过ActivityThread的performLaunchActivity()方法，使用ClassLoader对activity进行实例化。  
```java
try {
java.lang.ClassLoader cl = appContext.getClassLoader();
activity = mInstrumentation.newActivity(
cl, component.getClassName(), r.intent);
StrictMode.incrementExpectedActivityCount(activity.getClass());
r.intent.setExtrasClassLoader(cl);
r.intent.prepareToEnterProcess();
if (r.state != null) {
r.state.setClassLoader(cl);
}
```
之后调用该activity的attach()方法，实现widnow的实例化(部分源码)  
```java
final void attach() {
attachBaseContext(context);

mFragments.attachHost(null /*parent*/);

mWindow = new PhoneWindow(this, window, activityConfigCallback);
mWindow.setWindowControllerCallback(this);
mWindow.setCallback(this);
mWindow.setOnWindowDismissedCallback(this);
mWindow.getLayoutInflater().setPrivateFactory(this);
if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
mWindow.setSoftInputMode(info.softInputMode);
}
if (info.uiOptions != 0) {
mWindow.setUiOptions(info.uiOptions);
}
mUiThread = Thread.currentThread();
}
```
总结，看下这张图：  
(图片)  

Activity通过setContentView设置需要展示的布局，window(实际是PhoneWindow)处理整个布局的层级和数据，window是布局View的实际持有者，windowManger将布局添加到window添加到系统，window在加载过程中可以分为以下3种：  
1.应用窗口：所谓应用窗口指的就是该窗口对应一个Activity。  
2.子窗口：所谓子窗口指的是必须依附在某个父窗口之上，Dialog。  
3.系统窗口：不依赖于任何应用或者不依附在任何父窗口之上，如：Toast,来电窗口等。  
WindowManagerGlobal是windowManger的实际执行者，在整个应用中是单例模式，存放着所有View的数据，Activity在初始化的过程中，同样会实例化本地的window对象，实现Activity和window的关联,activity中使用的windowManager为window所持有。  



参考：《Android开发艺术探索》

