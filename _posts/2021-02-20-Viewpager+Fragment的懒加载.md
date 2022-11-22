---
layout: post
title: Viewpager+Fragment的懒加载
date: 2021-02-20 13:58:20
tags: [编程]
toc:  true
---
Viewpager+FragmentPagerAdapter/FragmentStatePagerAdapter实现滚动视图，在App中是很常见的设计，有时候在Fragment页面较多的情况下，需要在页面可见的情况，才对当前Fragment进行加载，也就是常说的懒加载。  
这里直接谈谈现在如今的方案：  
FragmentPagerAdapter构造函数增加了behavior参数，假如设置为BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT参数，那么，当Viewpager滑动的时候，只有当前正在显示的Fragment会回调onResume()方法。即可以直接在onResume()函数中进行数据加载和其他初始化的操作，很方便。  
它是怎么做到的呢？  
在AndroidX下，源码上针对FragmentTransaction做了部分修改，增加了setMaxLifecycle函数，该函数可以设置Fragent的生命周期的状态。  
ViewPager在滚动的时候，调用setPrimaryItem方法：  
```java
@Override
    public void setPrimaryItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
        Fragment fragment = (Fragment)object;
        if (fragment != mCurrentPrimaryItem) {
            if (mCurrentPrimaryItem != null) {
                mCurrentPrimaryItem.setMenuVisibility(false);
                if (mBehavior == BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
                    if (mCurTransaction == null) {
                        mCurTransaction = mFragmentManager.beginTransaction();
                    }
                    mCurTransaction.setMaxLifecycle(mCurrentPrimaryItem, Lifecycle.State.STARTED);
                } else {
                    mCurrentPrimaryItem.setUserVisibleHint(false);
                }
            }
            fragment.setMenuVisibility(true);
            if (mBehavior == BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
                if (mCurTransaction == null) {
                    mCurTransaction = mFragmentManager.beginTransaction();
                }
                mCurTransaction.setMaxLifecycle(fragment, Lifecycle.State.RESUMED);
            } else {
                fragment.setUserVisibleHint(true);
            }

            mCurrentPrimaryItem = fragment;
        }
    }
```
查看上述代码，可见，如果将behavior参数设置为BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT，会通过函数setMaxLifecycle，现将当前的Fragment设置为STARTED状态，再将需要显示的Fragment设置为RESUMED状态。因此只有当前的可见的Fragment才能回调onResume函数。