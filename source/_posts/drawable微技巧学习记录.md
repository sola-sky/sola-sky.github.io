---
title: drawable微技巧学习记录
date: 2016-05-22 21:12:01
tags: [drawable]
---

### mipmap

mipmap文件夹只是用来放置应用程序的icon的，仅此而已。
而且将icon放置在mipmap文件夹还可以让launcher图标自动拥有跨设备密度展示的能力，比如一台xxhdpi的设备可以自动加载mipmap-xxxhdpi下的icon来作为应用程序的luancher,这样图标看上去会更细腻些。
google也给出了各个密度下的icon尺寸设计：

| 密度 | 建议尺寸 |
|--------|--------|
|  mdpi       |    48*48    |
|  hdpi       |    72*72    |
|  xhdpi      |    96*96    |
|  xxhdpi     |    144*144  |
|  xxxhdpi    |    192*192  |

### 系统选择图片的过程

接下来看看各个dpi范围对应的密度吧

| dpi范围 | 密度 |
|--------|--------|
| 0 ~ 120dpi      |  ldpi      |
| 120 ~ 160dpi    |  mdpi      |
| 160 ~ 240dpi    |  hdpi      |
| 240 ~ 320dpi    |  xhdpi     |
| 320 ~ 480dpi    |  xxhdpi    |
| 480 ~ 640dpi    |  xxxhdpi   |

**drawable根据设备在相应的dpi下找图片，如有则正常按原尺寸显示，若没有则上一级dpi文件下找，若有则缩小显示，缩小的比例按dpi的来计算比如xxdpi设备找到xxxdpi的，因为xxdpi为3，xxxdpi为4，所以缩放0.75.**
我们当前的场景就是drawable-xxxhdpi文件夹，然后发现这里也没有android_logo这张图，接下来会尝试再找更高密度的文件夹，发现没有更高密度的了，这个时候会去**drawable-nodpi**文件夹找这张图，发现也没有，那么就会去更低密度的文件夹下面找，**依次是drawable-xhdpi -> drawable-hdpi -> drawable-mdpi -> drawable-ldpi**。
**drawable-nodpi文件夹是一个与密度无关的文件夹，放在这里的图片系统不会对其进行自动缩放，原始图片有多大就显示多大。**

**在放置图片的时候应该尽可能将图片放在高密度的dpi文件下**，因为如果本身设备为xxhdpi，但是图片却在较之低密度的文件时，根据上面的缩放比例，可想而知是大于1的，所以图片将会被放大，这**样就意味着像素变多，那样内存消耗将变大**。

### 源码角度解释

```java
final TypedValue value = new TypedValue();
is = res.openRawResource(id, value);
bm = decodeResourceStream(res, value, is, null, opts);
```

1. 原始资源，这个调用了 Resource.openRawResource 方法，这个方法调用完成之后会对 TypedValue 进行赋值，其中包含了原始资源的 density 等信息；原始资源的 density 其实取决于资源存放的目录（比如 drawable-xxhdpi 对应的是480， drawable-hdpi对应的就是240，而drawable目录对应的是TypedValue.DENSITY_DEFAULT=0） 
2. 调用 decodeResourceStream 对原始资源进行解码和适配。这个过程实际上就是原始资源的 density 到屏幕 density 的一个映射。

```java
public static Bitmap decodeResourceStream(Resources res, TypedValue value,
            InputStream is, Rect pad, Options opts) {
        if (opts == null) {
            opts = new Options();
        }
        if (opts.inDensity == 0 && value != null) {
            final int density = value.density;
            if (density == TypedValue.DENSITY_DEFAULT) {
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }
        if (opts.inTargetDensity == 0 && res != null) {
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }
        return decodeStream(is, pad, opts);
    }
```

该方法主要就是对opts对象中的属性进行赋值，代码不难理解。如果`alue.density == TypedValue.DENSITY_DEFAULT`就是0的话，将 **opts.inDensity赋值为 DisplayMetrics.DENSITY_DEFAULT默认值为160**.否则就将 opts.inDensity赋值为第一步获取到的值。**此外将 opts.inTargetDensity赋值为屏幕密度Dpi**。inDensity 和 inTargetDensity要特别注意，这两个值与下面 cpp 文件里面的 density 和 targetDensity 相对应。

```java
static jobject doDecode(JNIEnv* env, SkStreamRewindable* stream, jobject padding, jobject options) {
......
    if (env->GetBooleanField(options, gOptions_scaledFieldID)) {
        const int density = env->GetIntField(options, gOptions_densityFieldID);//通过JNI获取opts.inDensity的值
        const int targetDensity = env->GetIntField(options, gOptions_targetDensityFieldID);//通过JNI获取opts.inTargetDensity的值
        const int screenDensity = env->GetIntField(options, gOptions_screenDensityFieldID);
        if (density != 0 && targetDensity != 0 && density != screenDensity) {
            scale = (float) targetDensity / density;//求出缩放的倍数。
        }
    }
}
const bool willScale = scale != 1.0f;
......
SkBitmap decodingBitmap;
if (!decoder->decode(stream, &decodingBitmap, prefColorType,decodeMode)) {
   return nullObjectReturn("decoder->decode returned false");
}
//这里这个deodingBitmap就是解码出来的bitmap，大小是图片原始的大小
int scaledWidth = decodingBitmap.width();
int scaledHeight = decodingBitmap.height();
if (willScale && decodeMode != SkImageDecoder::kDecodeBounds_Mode) {
    scaledWidth = int(scaledWidth * scale + 0.5f);//缩放后的宽
    scaledHeight = int(scaledHeight * scale + 0.5f);//缩放后的高
}
if (willScale) {
    const float sx = scaledWidth / float(decodingBitmap.width());//宽的缩放倍数
    const float sy = scaledHeight / float(decodingBitmap.height());//高的缩放倍数
    ......
    SkPaint paint;
    SkCanvas canvas(*outputBitmap);
    canvas.scale(sx, sy);//缩放画布
    canvas.drawBitmap(decodingBitmap, 0.0f, 0.0f, &paint);//画出图像
}
......
}
```
关键代码为`scale = (float) targetDensity / density`，后面的代码都是根据该比例求出缩放后的图片高宽，然后将画布缩放至
```java
    const float sx = scaledWidth / float(decodingBitmap.width());//宽的缩放倍数
    const float sy = scaledHeight / float(decodingBitmap.height());//高的缩放倍数
    ......
    SkPaint paint;
    SkCanvas canvas(*outputBitmap);
    canvas.scale(sx, sy);//缩放画布
```

参考博客：

[王瑞刚的那些值得你去细细研究的Drawable适配](http://blog.csdn.net/wrg_20100512/article/details/51295317)
[郭霖的Android drawable微技巧，你所不知道的drawable的那些细节](Android drawable微技巧，你所不知道的drawable的那些细节)
