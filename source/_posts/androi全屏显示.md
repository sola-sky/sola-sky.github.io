---
title: android全屏显示
date: 2016-06-20 19:15:25
tags: FULL_SCREEN
---

### 第一种方式

通过`WindowManager.LayoutParams.FLAG_FULLSCREEN`来实现全屏功能
```java
getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
```

### 第二种方式 ###

从3.0版本之后，系统DecorView提供了一个`setSystemUiVisibility()`方法，可以通过设置Flag改变SystemUI的属性

从19之后，为`setSystemUiVisibility()`引入了新的标签`SYSTEM_UI_FLAG_IMMERSIVE`和`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`，标签与`SYSTEM_UI_FLAG_HIDE_NAVIGATION`和`SYSTEM_UI_FLAG_FULLSCREEN`一起使用的时候可以达到全屏的效果。

###### SYSTEM_UI_FLAG_IMMERSIVE ######

用户可以通过在边缘区域向内滑动来让系统栏重新显示,并清空`SYSTEM_UI_FLAG_HIDE_NAVIGATION`和`SYSTEM_UI_FLAG_FULLSCREEN`标签，还会触发`View.OnSystemUiVisibilityChangeListener`。系统栏显示后就不会自动隐藏了，必须通过手动设置。

###### SYSTEM_UI_FLAG_IMMERSIVE_STICKY ######

当使用了SYSTEM_UI_FLAG_IMMERSIVE_STICKY标签的时候，向内滑动的操作会让系统栏临时显示，并处于半透明的状态。此时没有标签会被清除，系统UI可见性监听器也不会被触发。如果用户没有进行操作，系统栏会在一段时间内自动隐藏。因为，因为在这个模式下展示的系统栏是处于暂时(transient)的状态。

###### 其他一些标签

- View.SYSTEM_UI_FLAG_LAYOUT_STABLE
- View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
- View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN

标签可以使当系统UI出现时不会导致Activity UI重新layout。

**注意：目前测试发现，当全屏后，按下电源键待机后再点亮屏幕，发现全屏有问题，不会实现全屏，而是没有全屏之前的设置。**

#### 参考文章
- [全屏沉浸式](http://hukai.me/android-training-course-in-chinese/ui/system-ui/immersive.html)
- [与Status Bar和Navigation Bar相关的一些东西](http://angeldevil.me/2014/09/02/About-Status-Bar-and-Navigation-Bar/)