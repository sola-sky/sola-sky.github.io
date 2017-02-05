---
title: JNI/NDK开发
date: 2017-02-04 10:54:02
tags: JNI 
---

# JNI/NDK开发

项目中使用了音视频技术，因此不可避免的需要用到jni技术，即java层调用native的代码，下面将介绍一些相关内容，并且有关代码直接采用项目中的例子。

## JNI/NDK开发流程  

### 1. 使用CMake构建NDK

1. 在项目的根目录下(也非一定要根目录，只要后面跟build.gradle中配置的路径相对应就行)创建一个CMakeLists.txt文件，**名字是固定只能是CMakeLists.txt**。

2. 在build.gradle中配置

   1. 在defaultConfig中配置类似的如下代码

      ```
      externalNativeBuild {
        cmake {
          cppFlags ""
          abiFilters 'x86', 'armeabi'//根据需要的平台进行添加
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

      **其中path对应于CMakeLists.txt的路径(这里用的是相对路径)**。

   3. 创建一个用于存放原生源文件的目录如cpp/目录

   **具体的内容可以参考如下：**

   [向您的项目添加 C 和 C++ 代码](https://developer.android.com/studio/projects/add-native-code.html)


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

主要是用于联系在Java类中声明的native方法，这样在Java层调用native方法时就能找到其相关的实现。

这里只介绍第二种方式。

在存放原生源文件的目录如cpp/目录下创建并编写相关的C/C++文件

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

将Java类中声明的native方法全部列出来，放到`JNINativeMethod`结构体类型的数组中，`JNINativeMethod`结构体的定义如下：

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

经过这一步之后，Java层中的native方法就能正确找到native层的函数实现了。

**可以参考如下链接：**

[JVM查找java native方法的规则](http://blog.csdn.net/xyang81/article/details/41854185)

### 4.编写具体的CMakeLists.txt内容

CMakeLists.txt的作用是用于配置如何产生想要的库，这里简单贴一下播放器部分的CMakeLists.txt内容

```cmake
# 这里是固定的要写版本
cmake_minimum_required(VERSION 3.4.1) 
··· ··· 
# 添加当前目录下的所有源文件
aux_source_directory(. SOLA_PLAYER_LISTS)
··· ···
# 以动态库形式增加一个solaplayer库，将相关的源文件加入这个动态库
add_library(solaplayer SHARED ${SOLA_PLAYER_LISTS} j4a/j4a_base.cpp j4a/audio_track.cpp)
# 链接一些在solaplayer库中需要使用的其它库，就是在之前添加的那些源文件中需要用到的其它的库的函数、类等
target_link_libraries(solaplayer sdl ffmpeg ${jnigraphics-lib} yuv ${android-lib})
```

然后所有情况都正常的情况下只要在android studio一运行就会产生libsolaplayer.so库，这样在执行`System.loadLibrary("solaplayer")`时就可以加载这个库，而不会发生找不到的情况了。

具体的有关cmake的内容可以参考如下：

[CMake实践](http://sewm.pku.edu.cn/src/paradise/reference/CMake%20Practice.pdf)

[CMake官方文档](https://cmake.org/cmake-tutorial/)

## 本地数据类型及与Java数据类型的映射关系

### 1.基本类型

| Java基本类型    | 本地类型         | 描述                   |
| :---------- | ------------ | -------------------- |
| **char**    | **jcahr**    | **unsigned 16 bits** |
| **short**   | **jshort**   | **signed 16 bits**   |
| **int**     | **jint**     | **signed 32 bits**   |
| **long**    | **jlong**    | **signed 64 bits**   |
| **boolean** | **jboolean** | **unsigned 8 bits**  |
| **byte**    | **jbyte**    | **signed 8 bits**    |
| **float**   | **jfloat**   | **32 bits**          |
| **double**  | **jdouble**  | **64 bits**          |

### 2.引用数据类型

| Java类型        | 本地类型               |
| ------------- | ------------------ |
| **Object**    | **jobject**        |
| **Class**     | **jclass**         |
| **String**    | **jstring**        |
| **Object[]**  | **jobjectArray**   |
| **boolean[]** | **jbooleanArray**f |
| **byte[]**    | **jbyteArray**     |
| **char[]**    | **jcharArray**     |
| **short[]**   | **jshortArray**    |
| **int[]**     | **jintArray**      |
| **long[]**    | **jlongArray**     |
| **float[]**   | **jfloatArray**    |
| **double[]**  | **jdoubleArray**   |

**特别注意：**

1. 多维数组（包括二维数组）都是引用类型，需要使用objectArray类型来进行表示，如二维数组就是指向一位数组的数组。

### 3. 域描述符

1. 基本类型描述符

   | Java类型      | 描述符   |
   | ----------- | ----- |
   | **boolean** | **Z** |
   | **byte**    | **B** |
   | **char**    | **C** |
   | **short**   | **S** |
   | **int**     | **I** |
   | **long**    | **J** |
   | **float**   | **F** |
   | **double**  | **D** |

上文 `{"getCurrentPosition", "()J", (void *)GetCurrentPosition}`中的*J*就是指long类型的描述符了。

2. 引用类型描述符

   一般引用类型为(*L + 该类型类描述符 + ;*) （主要其中的的分号 **";"**，这是描述符的一部分，而不是分段的意思），而**类描述符**是类全限定名称将原来的.分隔符换成/分隔符。如Java中的java.lang.String的类描述符就是java/lang/String。所以其域描述符就为`Ljava/lang/String;`。

## JNI异常处理

这里就简介介绍一下项目里的一个函数（来自于B站ijkplayer的源码）

```C++
bool J4A_ExceptionCheck__catchAll(JNIEnv *env)
{
    if (env->ExceptionCheck()) {
        env->ExceptionDescribe();
        env->ExceptionClear();
        return true;
    }

    return false;
}
```

用`ExceptionCheck()`函数检查一下最近JNI调用是否发送了异常，如果有则返回JNI_TRUE，则会执行上面代码中`if`花括号中的语句；若没有异常则返回JNI_FALSE。当检查到有异常时，调用`ExceptionDescribe()`来打印相关异常的堆栈信息，然后再调用`ExceptionClear()`函数来清除异常堆栈信息的缓冲区（防止后续异常堆栈信息覆盖之前的异常信息）。

## 参考文献

[JNI/NDK开发指南](http://blog.csdn.net/column/details/blogjnindk.html)