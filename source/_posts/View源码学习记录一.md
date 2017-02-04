---
title: View源码学习记录一
date: 2016-06-06 07:16:04
tags:
---

###View类的注释文档

- set properties
- set focus
- set up listeners
- set visibility

`onFinishInflate()`
Called after a view and all of its children has been inflated from xml.

`onMeasure()`
Called to determine the size requirements for this view and all of its children.

`onLayout()`
`onSizeChanged()`
`onDraw()`
`onKeyDown()`
`onKeyUp()`
`onTrackballEvent()`
`onTouchEvent`
`onFocusChanged`
`onWindowFocusChanged`
`onAttachedToWindow`
`onDetachedFromWindow`
`onWindowVisibilityChanged`

**measured width 和 measured height**
These dimensions define how big a view wants to be within its parent.
The measured dimensions can be obtained by calling getMeasuredWidth() getMeasureHeight()

**width and height or drawing width and drawing height**
These dimensions define the acutal size of the view on screen, **at drawing time and after layout**.These values may,but do no have to, be different from the measured width and height.
(**注意：width和height的获取是在layout方法之后才能获取到的，本质上是右边减左边坐标，而坐标是在layout时确定的，还有就是width可以和measured width不一样**)

To measure its dimensions, a view takes into account its padding.(**也就是说，为了测量，padding也是需要考虑进入的，也就是说计算measured width时需要加上padding**)

*Event though a view can define a padding, it does not provide any support for margins. However, view groups provide sucah a support.*
**ViewGroup and ViewGroup.MarginLayoutParams**

 A parent view may call measure() more than once on its children. For example, the parent may measure each child once with unspecified dimensions to find out how big they want to be, then call measure() on them again with actual numbers if the sum of all the children's unconstrained sizes is too big or too small.

The measure pass uses two classes to communicate dimensions.The
**MeasureSpec** lass is used by views to tell their parents how they want to be measured and positioned. The base LayoutParams class just describes how big the view wants to be for both width and height.

<li>UNSPECIFIED: This is used by a parent to determine the desired dimension
of a child view. For example, a LinearLayout may call measure() on its child with the height set to UNSPECIFIED and a width of EXACTLY 240 to find out how tall the child view wants to be given a width of 240 pixels.
<li>EXACTLY: This is used by the parent to impose an exact size on the child. The child must use this size, and guarantee that all of its descendants will fit within this size.
<li>AT_MOST: This is used by the parent to impose a maximum size on the child. The child must guarantee that it and all of its descendants will fit within this size.

<li>**requestLayout**: 典型的用法是当view确信当前的边界不再适合它时，可以调用这个方法重新布局。</li>

If you set a background drawable for a View, then the View will draw it before calling back to its <code>onDraw()</code> method. The child drawing order can be overridden with ViewGroup#setChildrenDrawingOrderEnabled(boolean) custom child drawing order in a ViewGroup, and with #setZ(float) custom Z values set on Views.
也就说背景图片在回调`onDraw()`之前就会被画。并且绘画的顺序可以通过`setChildrenDrawingOrderEnabled()`自定义子view的顺序，通过设置z坐标。

** To force a view to draw, call invalidate()**

**边界改变用requestLayout,可见性改变用invalidate()**

**注意：整个视图树是单线程，所以当你调用任何方法和view时都必须在UI线程。
If you are doing work on other threads and want to update the state of a view from that thread, you should use a Handler.

focus是用来处理用户输入的
When in touch mode (see notes below) views indicate whether they still would like focus via {@link #isFocusableInTouchMode} and can change this via {@link #setFocusableInTouchMode(boolean)}.
也就是说用来判断在触摸时，该view是否会持有焦点。
`isFocusable()`是用来判断该view是否有获取焦点的能力
`setFocusable()`赋予该view拥有获取焦点的能力
`requestFocus()`让该view获取到焦点

For a touch capable device, once the user touches the screen, the device will enter touch mode.  From this point onward, only views for which {@link #isFocusableInTouchMode} is true will be focusable, such as text editing widgets. Other views that are touchable, like buttons, will not take focus when touched; they will only fire the on click listeners.
Any time a user hits a directional key, such as a D-pad direction, the view device will exit touch mode, and find a view to take focus, so that the user may resume interacting with the user interface without touching the screen again.
这段话的意思是说当设备进入触摸模式时，只有那些在触摸模式下有能力获得focus的控件才能获得focus，比如TextView。但是像Button默认就是不会获得焦点了在触摸时。
可用`isInTouchMode()`来查看当前是否处于触摸模式


### View源码
```java
@IntDef({VISIBLE, INVISIBLE, GONE})
@Retention(RetentionPolicy.SOURCE)
public @interface Visibility {}
```

![setVisibility](http://thumbnail0.baidupcs.com/thumbnail/1d14172118811b1a6de058025448980c?fid=2350953866-250528-639134407184938&time=1465614000&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-MKdEjJJIByrk01WugcLIDB%2BgPqY%3D&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=3768716526557097506&dp-callid=0&size=c710_u400&quality=100)

因为`public void setVisibility(@Visibility int visibility)`的参数被@Visibility标记了，所以只能用@IntDef里声明的那些变量来对参数进行赋值，不然会提示错误。这么做是用来代替枚举型，因为Enum通常需要两倍以上的存储空间。而在常量定义了之后，事实上我们已经可以使之作为对Enum的替代了，但是在实际的开发过程中写的代码如果换成了其他的变量名，编译器并不能够报错。所以就用了**@IntDef @Retention @interface**这种办法。

- RetentionPolicy.SOURCE —— 这种类型的Annotations只在源代码级别保留,编译时就会被忽略
- RetentionPolicy.CLASS —— 这种类型的Annotations编译时被保留,在class文件中存在,但JVM将会忽略
- RetentionPolicy.RUNTIME —— 这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使

### StateSet

State sets are arrays of positive ints where each element represents the state of a {@link android.view.View} (e.g. focused, selected, visible, etc.).  A {@link android.view.View} may be in one or more of those states.

每个元素代表View的状态，比如focused，selected，visible等。

A state spec is an array of signed ints where each element represents a required (if positive) or an undesired (if negative) {@link android.view.View} state.
元素为正代表需要，为负代表没有需要。

`private static final int[][] VIEW_STATE_SETS;`在相应的行存放不同状态（状态可以组合）的引用，比如WINDOW_FOCUSED是在index为1（坐标从0开始）的那行，及第二行了。比如3为WINDOW_FOCUSED和SELECTED的组合，一维数组的长度就是2了，分别就存放这两个状态对应的drawable的Int值了（R），至于第一是存放WINDOW_FOCUSED还是第二个存放它，就得看`R.styleable.ViewDrawableStates[]`数组中这两个状态是先后顺序了。

**View中的SYSTEM_UI_FLAG_ 这些属性标签对于控制屏幕显示挺重要的**
