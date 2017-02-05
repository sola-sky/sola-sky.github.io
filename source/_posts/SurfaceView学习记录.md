---
title: SurfaceView学习记录
date: 2016-06-22 19:03:03
tags: SurfaceView
---

[原文出处：SurfaceView，老罗的文章](http://blog.csdn.net/luoshengyang/article/details/8661317)

```java
    protected void onAttachedToWindow() {  
        super.onAttachedToWindow();  
        mParent.requestTransparentRegion(this);  
        mSession = getWindowSession();  
        ......  
    }  
```

第一件事情是通知父视图，当前正在处理的SurfaceView需要在宿主窗口的绘图表面上挖一个洞，即需要在宿主窗口的绘图表面上设置一块透明区域。当前正在处理的SurfaceView的父视图保存在父类View的成员变量mParent中，通过调用这个成员变量mParent所指向的一个ViewGroup对象的成员函数requestTransparentRegion，就可以通知到当前正在处理的SurfaceView的父视图，当前正在处理的SurfaceView需要在宿主窗口的绘图表面上设置一块透明区域。

第二件事情是调用从父类View继承下来的成员函数getWindowSession来获得一个实现了IWindowSession接口的Binder代理对象，并且将该Binder代理对象保存在SurfaceView类的成员变量mSession中。从前面Android应用程序窗口（Activity）实现框架简要介绍和学习计划这个系列的文章可以知道，在Android系统中，每一个应用程序进程都有一个实现了IWindowSession接口的Binder代理对象，这个Binder代理对象是用来与WindowManagerService服务进行通信的，View类的成员函数getWindowSession返回的就是该Binder代理对象。在接下来的Step 8中，我们就可以看到，SurfaceView就可以通过这个实现了IWindowSession接口的Binder代理对象来请求WindowManagerService服务为自己创建绘图表面的。


```
public class SurfaceView extends View {  
    ......  
  
    boolean mRequestedVisible = false;  
    boolean mWindowVisibility = false;  
    boolean mViewVisibility = false;  
    .....  
  
    @Override  
    protected void onWindowVisibilityChanged(int visibility) {  
        super.onWindowVisibilityChanged(visibility);  
        mWindowVisibility = visibility == VISIBLE;  
        mRequestedVisible = mWindowVisibility && mViewVisibility;  
        updateWindow(false, false);  
    }  
  
    ......  
}  
```

 SurfaceView类有三个用来描述可见性的成员变量mRequestedVisible、mWindowVisibility和mViewVisibility。其中，mWindowVisibility表示SurfaceView的宿主窗口的可见性，mViewVisibility表示SurfaceView自身的可见性。只有当mWindowVisibility和mViewVisibility的值均等于true的时候，mRequestedVisible的值才为true，表示SurfaceView是可见的。

绘图表面类型为SURFACE_TYPE_PUSH_BUFFERS的SurfaceView的UI是不能由应用程序来控制的，而是由专门的服务来控制的，例如，摄像头服务或者视频播放服务，同时，SurfaceFlinger服务会使用一种特殊的LayerBuffer来描述这种绘图表面。使用LayerBuffer来描述的绘图表面在进行渲染的时候，可以使用硬件加速，例如，使用copybit或者overlay来加快渲染速度，从而可以获得更流畅的摄像头预览或者视频播放。

- mHaveFrame，用来描述SurfaceView的宿主窗口的大小是否已经计算好了。只有当宿主窗口的大小计算之后，SurfaceView才可以更新自己的窗口。
- mRequestedWidth，用来描述SurfaceView最后一次被请求的宽度。

- mRequestedHeight，用来描述SurfaceView最后一次被请求的高度。

- mRequestedFormat，用来描述SurfaceView最后一次被请求的绘图表面的像素格式。

- mNewSurfaceNeeded，用来描述SurfaceView是否需要新创建一个绘图表面。

- mLeft、mTop、mWidth、mHeight，用来描述SurfaceView上一次所在的位置以及大小。

- mFormat，用来描述SurfaceView的绘图表面上一次所设置的格式。

- mVisible，用来描述SurfaceView上一次被设置的可见性。

- mType，用来描述SurfaceView的绘图表面上一次所设置的类型。

- mUpdateWindowNeeded，用来描述SurfaceView是否被WindowManagerService服务通知执行一次UI更新操作。

- mReportDrawNeeded，用来描述SurfaceView是否被WindowManagerService服务通知执行一次UI绘制操作。

- mLayout，指向的是一个WindowManager.LayoutParams对象，用来传递SurfaceView的布局参数以及属性值给WindowManagerService服务，以便WindowManagerService服务可以正确地维护它的状态。

### 挖洞过程
从SurfaceView的绘图表面的创建过程可以知道，SurfaceView在被附加到宿主窗口之上的时候，SurfaceView类的成员函数onAttachedToWindow就会被调用。SurfaceView类的成员函数onAttachedToWindow在被调用的期间，就会请求在宿主窗口上设置透明区域。

```
public class SurfaceView extends View {  
    ......  
  
    @Override  
    protected void onAttachedToWindow() {  
        super.onAttachedToWindow();  
        mParent.requestTransparentRegion(this);  
        ......  
    }  
  
    ......  
}  
```

```java
public abstract class ViewGroup extends View implements ViewParent, ViewManager {  
    ......  
  
    public void requestTransparentRegion(View child) {  
        if (child != null) {  
            child.mPrivateFlags |= View.REQUEST_TRANSPARENT_REGIONS;  
            if (mParent != null) {  
                mParent.requestTransparentRegion(this);  
            }  
        }  
    }  
  
    ......  
}  
```

```java
public final class ViewRoot extends Handler implements ViewParent,  
        View.AttachInfo.Callbacks {  
    ......  
  
    public void requestTransparentRegion(View child) {  
        // the test below should not fail unless someone is messing with us  
        checkThread();  
        if (mView == child) {  
            mView.mPrivateFlags |= View.REQUEST_TRANSPARENT_REGIONS;  
            // Need to make sure we re-evaluate the window attributes next  
            // time around, to ensure the window has the correct format.  
            mWindowAttributesChanged = true;  
            requestLayout();  
        }  
    }  
  
    ......  
}  
```

当一个窗口被请求设置了一块透明区域之后，它的窗口属性就发生变化了，因此，这时候除了要将与它所关联的一个ViewRoot对象的成员变量mWindowAttributesChanged的值设置为true之外，还要调用该ViewRoot对象的成员函数requestLayout来请求刷新一下窗口的UI，即请求对窗口的UI进行重新布局和绘制。
ViewRoot类的成员函数requestLayout最终会调用到另外一个成员函数performTraversals来实际执行刷新窗口UI的操作。
ViewRoot类的成员函数performTraversals在刷新窗口UI的过程中，就会将嵌入在它里面的SurfaceView所要设置的透明区域收集起来，以便可以请求WindowManagerService将这块透明区域设置到它的绘图表面上去。
**挖洞过程主要是使用Region，从顶层视图将其往子View传递Region，在传递过程中通过调用Region的op()方法，将某区域设置为Region.Op.DIFFERENCE或者为Region.Op.UNION来区分是否将其透明，前者不透明，后者透明处理(sWindowSession.setTransparentRegion()方法将UNION的区域设置)**

### 绘制过程

```
public class SurfaceView extends View {  
    ......  
  
    @Override  
    public void draw(Canvas canvas) {  
        if (mWindowType != WindowManager.LayoutParams.TYPE_APPLICATION_PANEL) {  
            // draw() is not called when SKIP_DRAW is set  
            if ((mPrivateFlags & SKIP_DRAW) == 0) {  
                // punch a whole in the view-hierarchy below us  
                canvas.drawColor(0, PorterDuff.Mode.CLEAR);  
            }  
        }  
        super.draw(canvas);  
    }  
  
    @Override  
    protected void dispatchDraw(Canvas canvas) {  
        if (mWindowType != WindowManager.LayoutParams.TYPE_APPLICATION_PANEL) {  
            // if SKIP_DRAW is cleared, draw() has already punched a hole  
            if ((mPrivateFlags & SKIP_DRAW) == SKIP_DRAW) {  
                // punch a whole in the view-hierarchy below us  
                canvas.drawColor(0, PorterDuff.Mode.CLEAR);  
            }  
        }  
        // reposition ourselves where the surface is   
        mHaveFrame = true;  
        updateWindow(false, false);  
        super.dispatchDraw(canvas);  
    }  
  
    ......  
}  
```
本来SurfaceView类的成员函数draw是用来将自己的UI绘制在宿主窗口的绘图表面上的，但是这里我们可以看到，如果当前正在处理的SurfaceView不是用作宿主窗口面板的时候，即其成员变量mWindowType的值不等于WindowManager.LayoutParams.TYPE_APPLICATION_PANEL的时候，SurfaceView类的成员函数draw只是简单地将它所占据的区域绘制为黑色。
本来SurfaceView类的成员函数dispatchDraw是用来绘制SurfaceView的子视图的，但是这里我们同样看到，如果当前正在处理的SurfaceView不是用作宿主窗口面板的时候，那么SurfaceView类的成员函数dispatchDraw只是简单地将它所占据的区域绘制为黑色，同时，它还会通过调用另外一个成员函数updateWindow更新自己的UI，实际上就是请求WindowManagerService服务对自己的UI进行布局，以及创建绘图表面。
从SurfaceView类的成员函数draw和dispatchDraw的实现就可以看出，SurfaceView在其宿主窗口的绘图表面上面所做的操作就是将自己所占据的区域绘为黑色，除此之外，就没有其它更多的操作了，这是因为SurfaceView的UI是要展现在它自己的绘图表面上面的。接下来我们就分析如何在SurfaceView的绘图表面上面进行UI绘制。
注意，只有在一个SurfaceView的绘图表面的类型不是SURFACE_TYPE_PUSH_BUFFERS的时候，我们才可以自由地在上面绘制UI。

在`SurfaceViewHolder.lockCanvas()`其内部实现时，在成功获取画布时，是调用了一个域来对画布进行锁定的。 `final ReentrantLock mSurfaceLock = new ReentrantLock(); `
具体代码如下:
```
public class SurfaceView extends View {  
    ......  
  
    final ReentrantLock mSurfaceLock = new ReentrantLock();  
    final Surface mSurface = new Surface();  
    ......  
  
    private SurfaceHolder mSurfaceHolder = new SurfaceHolder() {  
        ......  
  
        public Canvas lockCanvas() {  
            return internalLockCanvas(null);  
        }  
  
        ......  
  
        private final Canvas internalLockCanvas(Rect dirty) {  
            if (mType == SURFACE_TYPE_PUSH_BUFFERS) {  
                throw new BadSurfaceTypeException(  
                        "Surface type is SURFACE_TYPE_PUSH_BUFFERS");  
            }  
            mSurfaceLock.lock();  
            ......  
  
            Canvas c = null;  
            if (!mDrawingStopped && mWindow != null) {  
                Rect frame = dirty != null ? dirty : mSurfaceFrame;  
                try {  
                    c = mSurface.lockCanvas(frame);  
                } catch (Exception e) {  
                    Log.e(LOG_TAG, "Exception locking surface", e);  
                }  
            }  
            ......  
  
            if (c != null) {  
                mLastLockTime = SystemClock.uptimeMillis();  
                return c;  
            }  
            ......  
  
            mSurfaceLock.unlock();  
  
            return null;    
        }   
  
        ......           
    }  
  
    ......  
}  
``` 

### 总结
SurfaceView有以下三个特点：
- 具有独立的绘图表面；
- 需要在宿主窗口上挖一个洞来显示自己；
- 它的UI绘制可以在独立的线程中进行，这样就可以进行复杂的UI绘制，并且不会影响应用程序的主线程响应用户输入。