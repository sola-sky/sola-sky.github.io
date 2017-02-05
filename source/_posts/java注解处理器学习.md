---
title: java注解处理器学习
date: 2016-06-20 20:53:25
tags: Annotation
---

#### 参考：
- [butterknife 源码分析](http://2dxgujun.com/post/2015/06/07/butterknife-analysis.html)
- [最新ButterKnife框架原理](https://bxbxbai.github.io/2016/03/12/how-butterknife-works/)
- [Java注解处理器](http://www.race604.com/annotation-processing/)

##### AbstractProcessor

- `init(ProcessingEnvironment env)`: 这里有一个特殊的init()方法，它会被注解处理工具调用，并输入ProcessingEnviroment参数。ProcessingEnviroment提供很多有用的工具类Elements, Types和Filer。
- `process(Set<? extends TypeElement> annotations, RoundEnvironment env)`: 这相当于每个处理器的主函数main()。你在这里写你的扫描、评估和处理注解的代码，以及生成Java文件。输入参数RoundEnviroment，可以让你查询出包含特定注解的被注解元素。

##### 注册你的处理器

你可能会问，我怎样将处理器MyProcessor注册到javac中。你必须提供一个.jar文件。就像其他.jar文件一样，你打包你的注解处理器到此文件中。并且，在你的jar中，你需要打包一个特定的文件javax.annotation.processing.Processor到META-INF/services路径下.
打包进MyProcessor.jar中的javax.annotation.processing.Processor的内容是，注解处理器的合法的全名列表，每一个元素换行分割:

```java
com.example.MyProcessor
com.foo.OtherProcessor
net.blabla.SpecialProcessor
```

把MyProcessor.jar放到你的builpath中，javac会自动检查和读取javax.annotation.processing.Processor中的内容，并且注册MyProcessor作为注解处理器。


- @Target(ElementType.TYPE)   //接口、类、枚举、注解
- @Target(ElementType.FIELD) //字段、枚举的常量
- @Target(ElementType.METHOD) //方法
- @Target(ElementType.PARAMETER) //方法参数
- @Target(ElementType.CONSTRUCTOR)  //构造函数
- @Target(ElementType.LOCAL_VARIABLE)//局部变量
- @Target(ElementType.ANNOTATION_TYPE)//注解
- @Target(ElementType.PACKAGE) ///包   
