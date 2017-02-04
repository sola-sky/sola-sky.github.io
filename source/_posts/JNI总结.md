---
title: JNI/NDK开发
date: 2017-02-04 10:54:02
tags: JNI 
---

# JNI/NDK开发

项目中使用了音视频技术，因此不可避免的需要用到jni技术，即java层调用c/c++层的代码，下面将介绍一些相关内容，并且有关代码直接采用项目中的例子。

## JNI/NDK开发流程  

### 1. 使用CMake构建NDK

1. 在项目的根目录下(也非一定要根目录，只要后面跟build.gradle中配置的路径相对应就行)创建一个CMakeLists.txt文件，名字是固定只能是CMakeLists.txt

2. 在build.gradle中配置CMake

   1. 在defaultConfig中配置类似的如下代码

      ```
      externalNativeBuild {
        cmake {
          cppFlags ""
          abiFilters 'x86', 'armeabi'
        }
      }
      ```

   2. 在android中配置如下类似代码

      ```
      externalNativeBuild {
        cmake {
          path "CMakeLists.txt"
        }
      }
      ```

      其中path对应于CMakeLists.txt的路径

   3. 创建一个用于存放原生源文件的目录如cpp/目录


### 2. 编写声明了native方法的Java类

```java
public class SolaMediaPlayer extends AbstractMediaPlayer {
  ··· ···
  //这部分代码为了演示进行了更改，跟项目里的不一致
  static {
    System.loadLibrary("solaplayer"); //加载实现了native函数的动态库
  }
  ··· ···
  @Override
  public native long getCurrentPosition(); //声明这是一个native函数，由本地代码实现
  ··· ···
}
```

### 3. 实现JVM查找native方法

JVM查找native方法有两种方式：

1. 按照JNI命名规范的命名规则
2. 调用JNI提供的`RegisterNatives()`函数，将本地函数注册到JVM中  

本人倾向使用第二种方法，所以这里只介绍第二种。

在创建好的cpp/目录下创建并编写相关的C/C++文件

``` C++
··· ···
static JNINativeMethod g_methods[] = {
        ··· ···
        {"_pause", "()V", (void *)Pause},
        {"_stop", "()V", (void *)Stop},
        ··· ···
        {"_flush", "()V", (void *)Flush},
        {"getCurrentPosition", "()J", (void *)GetCurrentPosition},
        ··· ···
};
··· ···
```

将第一步Java类中声明的native方法全部列出来，放到`JNINativeMethod`结构体类型的数组中，`JNINativeMethod`结构体的定义如下：

``` C++
typedef struct {
    const char* name;//对应Java类中native方法的名称，如之前的getCurrentPosition
    const char* signature; //相应方法的签名，如之前的()J,括号里的为name方法的参数类型，由于getCurrentPosition方法没有参数所以不用写，J为其返回类型，因为getCurrentPosition返回long类型，在JNI中long对应的就是J,这个后面会介绍
    void* fnPtr; //本地函数，实现Java层的native方法
} JNINativeMethod;
```

也就是说这个结构体将Java层的方法对应到了C/C++层的函数，当在Java层调用name方法时，其实调用的是C/C++层中的fnPtr函数。

接下来就是实现相关的本地函数了，如`GetCurrentPosition`：

``` C++
jlong GetCurrentPosition(JNIEnv *env, jobject thiz) {
  jlong retval = 0;
  ··· ···
  return retval;
}
```

最后我们需要将本地函数注册到JVM中，由于JNI组件被成功加载和卸载时，会进行函数回调，当VM执行到`System.loadLibrary(xxx)`方法时，会去执行JNI组件中的`JNI_OnLoad()`函数，而在VM释放组件时会调用`JNI_OnUnload()`函数。所以我们可以在`JNI_OnLoad()`中进行本地函数注册。  

```C++
JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env = NULL;
    ··· ···
    if (vm->GetEnv((void **)&env, JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }
    ··· ···
    const char *class_name = "com/apical/av/SolaMediaPlayer"; //Java类的全限定名称,将.改成/
    g_clazz.clazz = env->FindClass(class_name); //找到相应Java类对应的jclass对象
    // 通过RegisterNatives将本地函数注册到JVM中，然后Java层就可以调用C/C++层对应的函数
    if (env->RegisterNatives(g_clazz.clazz, g_methods , sizeof(g_methods) / sizeof(g_methods[0])) < 0) {
        env->DeleteLocalRef(g_clazz.clazz);
        return -1;
    }
    ··· ···
    return JNI_VERSION_1_6;
}
```

### 4.编写具体的CMakeLists.txt内容









## JNI数据类型及与Java数据类型的映射关系



## JNI异常处理



