---
title: Annotation学习
date: 2016-06-12 20:07:07
tags:
---

### Annotation
- [参考一](http://www.cnblogs.com/mandroid/archive/2011/07/18/2109829.html)
- [参考二](http://a.codekk.com/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E6%B3%A8%E8%A7%A3%20Annotation)

#### 原理

Annotation其实是一种接口。通过Java的反射机制相关的API来访问annotation信息。相关类（框架或工具中的类）根据这些信息来决定如何使用该程序元素或改变它们的行为。

Java语言解释器在工作时会忽略这些annotation，因此在JVM	中这些annotation是“不起作用”的，只能通过配套的工具才能对这些annontaion类型的信息进行访问和处理。

##### Annotation与interface的异同

- Annotation类型使用关键字@interface而不是interface。这个关键字声明隐含了一个信息：它是继承java.lang.annotation.Annotation接口，并非声明了一个interface。
- Annotation 类型的方法必须声明为无参数、无异常抛出的。这些方法定义了annotation的成员：方法名成为了成员名，而方法返回值成为了成员的类型。方法的后面可以使用 default和一个默认数值来声明成员的默认值，null不能作为成员默认值。

#### 元Annotation

- @Documented 是否会保存到 Javadoc 文档中
- @Retention 保留时间，可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS，SOURCE 大都为 Mark Annotation，这类 Annotation 大都用来校验，比如 Override, SuppressWarnings
- @Target 可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER 等，未标注则表示可修饰所有
- @Inherited 是否可以被继承，默认为 false

#### 自定义

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface MethodInfo {

    String author() default "trinea@gmail.com";

    String date();

    int version() default 1;
}
```

当annotation只有单一成员，并成员命名为"value",这时可以省去"value="。上述的代码中由于不是这种情况，所以运用是得如下：

```java
@MethodInfo(author="Li Minyi", date="2016/06/12, version=1)
public String getInfo() {
return null;
}
```

如果将`MethodInfo`改成如下这样：

```java
public @interface MethodInfo {
    String value();
}
```

那么运用的时候可以不同value=,可以直接如下:

```java
@MethodInfo("Li Minyi") {
public String getInfo() {
return null;
}
```

注意观察MethodInfo后面括号中的内容。
**上述情况必须是单一成员，并且命名为value，返回类型随意**

稍微总结一下：
- 方法名成为了成员名，而方法返回值成为了成员的类型。
- 方法返回值只能是基本类型，String, Class, annotation, enumeration 或者是他们的一维数组

#### Annotation解析

```java
method.getAnnotation(AnnotationName.class);
method.getAnnotations();
method.isAnnotationPresent(AnnotationName.class);
```

- getAnnotation(AnnotationName.class) 表示得到该 Target 某个 Annotation 的信息，因为一个 Target 可以被多个 Annotation 修饰,比如`@Override@MethIfo`修饰一个方法。
- getAnnotations() 则表示得到该 Target 所有 Annotation
- isAnnotationPresent(AnnotationName.class) 表示该 Target 是否被某个 Annotation 修饰


