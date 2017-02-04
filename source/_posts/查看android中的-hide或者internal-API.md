---
title: 查看android中的@hide或者internal API
date: 2016-05-30 07:28:42
tags: @hide internal
---

### 对张鸿洋《Android轻松的查看与使用hide与internal API》的学习记录

在android开发中，我们会指定编译版本，这个版本会对应使用我们下载的SDK目录中对应版本的android.jar，这这个jar中，默认移除了所有被'@hide'标识的方法或者类，以及internal包下的类。

既然被去掉，那我们加上就好了

- 从android设备中提取(`/system/framework/framework.jar`)
[看这篇文章](http://blog.csdn.net/linghu_java/article/details/8283042)

- 从一些提供源码下载的网站下载(http://grepcode.com/)
[查看这篇文章](http://blog.csdn.net/liu470368500/article/details/40189397)

- 从github上获取完整的android.jar
[相关地址](https://github.com/anggrayudi/android-hidden-api)
我们只要用其中的android.jar替换掉相应目录下的android.jar就行了。

**不需要反射轻松调用被`@hide`标识的类或者方法，以及在internal下的类，可以直接创建，不过，不管是@hide还是internal包下的API，只要使用了，都有不确定的因素，说不定哪天该方法该类就不存在了。**
