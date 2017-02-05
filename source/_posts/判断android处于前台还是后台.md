---
title: 判断android处于前台还是后台
date: 2016-05-25 19:56:35
tags:
---

### 方法一 通过RunningTask

###### 原理

当一个App处于前台的时候，会处于RunningTask的这个栈的栈顶，所以我们可以取出RunningTask的栈顶的任务进程，看他与我们的想要判断的App的包名是否相同，来达到效果。

###### 缺点

getRunningTask方法在Android5.0以上已经被废弃，只会返回自己和系统的一些不敏感的task，不再返回其他应用的task，用此方法来判断自身App是否处于后台，仍然是有效的，但是无法判断其他应用是否位于前台，因为不再能获取信息。

###### 用法

```java
 List<ActivityManager.RunningTaskInfo> taskInfo = am.getRunningTasks(1);
            ComponentName componentInfo = taskInfo.get(0).topActivity;
            if (componentInfo.getPackageName().equals(context.getPackageName())) {
                isInBackground = false;
            }
```

### 方法二 通过RunningProcess

###### 原理

通过runningProcess获取到一个当前正在运行的进程的List，我们遍历这个List中的每一个进程，判断这个进程的一个importance 属性是否是前台进程，并且包名是否与我们判断的APP的包名一样，如果这两个条件都符合，那么这个App就处于前台

###### 缺点

在聊天类型的App中，常常需要常驻后台来不间断的获取服务器的消息，这就需要我们把Service设置成START_STICKY，kill 后会被重启（等待5秒左右）来保证Service常驻后台。如果Service设置了这个属性，这个App的进程就会被判断是前台，代码上的表现就是appProcess.importance的值永远是 ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND，这样就永远无法判断出到底哪个是前台了。

###### 用法

```java
 List<ActivityManager.RunningAppProcessInfo> runningProcesses = am.getRunningAppProcesses();
            for (ActivityManager.RunningAppProcessInfo processInfo : runningProcesses) {
                if (processInfo.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {
                    for (String activeProcess : processInfo.pkgList) {
                        if (activeProcess.equals(context.getPackageName())) {
                            isInBackground = false;
                        }
                    }
                }
            }
```

### 方法三 通过ActivityLifecycleCallbacks

###### 原理
AndroidSDK14在Application类里增加了ActivityLifecycleCallbacks，我们可以通过这个Callback拿到App所有Activity的生命周期回调。
可能还有人在纠结，我用back键切到后台和用Home键切到后台，一样吗？以上方法适用吗？在Android应用开发中一般认为back键是可以捕获的，而Home键是不能捕获的（除非修改framework）,但是上述方法从Activity生命周期着手解决问题，虽然这两种方式的Activity生命周期并不相同，但是二者都会执行onStop（）；所以并不关心到底是触发了哪个键切入后台的。另外,Application是否被销毁,都不会影响判断的正确性

###### 用法

```java
class MyApplication extends Application {
    public void onCreate(){
        Foreground.init(this);
    }
}

public interface ActivityLifecycleCallbacks {
        void onActivityCreated(Activity activity, Bundle savedInstanceState);
        void onActivityStarted(Activity activity);
        void onActivityResumed(Activity activity);
        void onActivityPaused(Activity activity);
        void onActivityStopped(Activity activity);
        void onActivitySaveInstanceState(Activity activity, Bundle outState);
        void onActivityDestroyed(Activity activity);
    }
```

### 方法四 通过使用UsageStatsManager获取

####### 原理

通过使用UsageStatsManager获取，此方法是Android5.0之后提供的新API，可以获取一个时间段内的应用统计信息，但是必须满足一下要求

- 此方法只在android5.0以上有效
- AndroidManifest中加入此权限 &lt;uses-permission xmlns:tools="http://schemas.android.com/tools" android:name="android.permission.PACKAGE_USAGE_STATS" tools:ignore="ProtectedPermissions" /&gt;

### 方法五 通过Android自带的无障碍功能

###### 原理

Android 辅助功能(AccessibilityService) 为我们提供了一系列的事件回调，帮助我们指示一些用户界面的状态变化。 我们可以派生辅助功能类，进而对不同的 AccessibilityEvent 进行处理。 同样的，这个服务就可以用来判断当前的前台应用。

###### 优势

1. AccessibilityService 有非常广泛的 ROM 覆盖，特别是非国产手机，从 Android API Level 8(Android 2.2) 到 Android Api Level 23(Android 6.0)。
2. AccessibilityService 不再需要轮询的判断当前的应用是不是在前台，系统会在窗口状态发生变化的时候主动回调，耗时和资源消耗都极小。
3. 不需要权限请求。
4. 它是一个稳定的方法，与 “方法6”读取 /proc 目录不同，它并非利用 Android 一些设计上的漏洞，可以长期使用的可能很大。
5. 可以用来判断任意应用甚至 Activity, PopupWindow, Dialog 对象是否处于前台。

###### 劣势

1. 需要要用户开启辅助功能。
2. 辅助功能会伴随应用被“强行停止”而剥夺。

### 方法六 读取Linux系统内核保存在/proc目录下的process进程信息

###### 原理

无意中看到乌云上有人提的一个漏洞，Linux系统内核会把process进程信息保存在/proc目录下，Shell命令去获取的他，再根据进程的属性判断是否为前台。

###### 优点

1. 不需要任何权限。
2. 可以判断任意一个应用是否在前台，而不局限在自身应用。

###### 缺点

1. 当/proc下文件夹过多时,此方法是耗时操作

###### 用法

获取一系列正在运行的App的进程

```java
List<AndroidAppProcess> processes = ProcessManager.getRunningAppProcesses();
```

获取任一正在运行的App进程的详细信息

```java
AndroidAppProcess process = processes.get(location);
String processName = process.name;

Stat stat = process.stat();
int pid = stat.getPid();
int parentProcessId = stat.ppid();
long startTime = stat.stime();
int policy = stat.policy();
char state = stat.state();

Statm statm = process.statm();
long totalSizeOfProcess = statm.getSize();
long residentSetSize = statm.getResidentSetSize();

PackageInfo packageInfo = process.getPackageInfo(context, 0);
String appName = packageInfo.applicationInfo.loadLabel(pm).toString();
```

判断是否在前台

```java
if (ProcessManager.isMyProcessInTheForeground()) {
  // do stuff
}
```

获取一系列正在运行的App进程的详细信息

```java
List<ActivityManager.RunningAppProcessInfo> processes = ProcessManager.getRunningAppProcessInfo(ctx);
```

#### 来源
- [wenmingvs/AndroidProcess](https://github.com/wenmingvs/AndroidProcess)
- [Is my Android app currently foreground or background](http://http://steveliles.github.io/is_my_android_app_currently_foreground_or_background.html)
- [Android How to Check if the App is in Background or Foreground](http://tips.androidhive.info/2015/04/android-how-to-check-if-the-app-is-in-background-or-foreground/)
