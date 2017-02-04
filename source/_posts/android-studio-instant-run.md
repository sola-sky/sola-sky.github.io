---
title: android studio instant run
date: 2016-05-31 07:45:23
tags:
---

###记录有关Instant Run的学习
[郭霖的原文——《你真的了解Instant Run吗？》](http://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650236001&idx=1&sn=f2ac9a45ebe0d59fa11d9599ad7cca50&scene=4#wechat_redirect)

Instant Run主要分为三种类型，hot swap、warm swap和cold swap，Android Studio会根据代码的修改情况自动选择使用哪种swap类型。

- **Hot Swap**

hot swap的适用条件比较少，只有一种情况会被Android Studio视为hot swap类型，就是**修改一个现有方法中的代码** **

- **Warm Swap**

warm swap的适用条件也比较局限，只有一种情况会被Android Studio视为warm swap类型，就是**修改或删除一个现有的资源文件**

- **Cold Swap**

cold swap相对而言就要更慢一些了，Android Studio会自动记录我们项目的每次修改，然后将修改的这部分内容打成一个dex文件发送到手机上，尽管这种swap类型仍然不需要去安装一个全新的APK，但是为了加载这个新的dex文件，整个应用程序必须进行重启才行。另外，cold swap的工作原理是基于multidex机制来实现的，在不引入外部library的情况下，只有5.0及以上的设备才支持multidex，因此，如果你使用了5.0以下的设备，那么cold swap就无法工作了，这种情况会执行最原始的完整APK安装过程。
cold swap的适用条件非常多。

- 添加、删除或修改一个注解
- 添加、删除或修改一个字段
- 添加、删除或修改一个方法
- 添加一个类
- 修改一个类的继承结构
- 修改一个类的接口实现
- 修改一个类的static修饰符
- 涉及资源文件id的改动


- **Full APK**

除了满足以上条件的其他程序变更，Instant Run目前都还不支持，主要包括以下一些情况：

- 改变AndroidManifest.xml文件的内容
- 改变被AndroidManifest.xml文件所引用的资源，比如string.xml中的app_name
- 改变桌面widget的UI相关元素

当程序不被Instant Run所支持时，就会执行完整APK安装过程，同时Android Studio会给出这样的提示：
**Performing full build & install**


