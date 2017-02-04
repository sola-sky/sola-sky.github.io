---
title: View源码学习二
date: 2016-06-14 07:07:35
tags:
---

# View

#### 重要Field

- protected ViewParent mParent
- AttachInfo mAttachInfo
- protected ViewGroup.LayoutParams mLayoutParams
- TransformationInfo mTransformationInfo
- private StateListAnimator mStateListAnimator
- private PerformClick mPerformClick
- public static final int LAYER_TYPE_NONE
- final RenderNode mRenderNode(感觉很重要)


### 重要的内部类

- TintInfo(主要包含 ColorStateList PorterDuff.Mode)

##### ListenerInfo
###### OnFocusChangeListener
protected OnFocusChangeListener mOnFocusChangeListener

- void onFocusChange(View v, boolean hasFocus)

###### OnLayoutChangeListener
private ArrayList&lt;OnLayoutChangeListener&gt; mOnLayoutChangeListeners;

- onLayoutChange

###### OnGenericMotionListener
用来处理鼠标滚轮

- onGenericMotion

###### OnHoverListener
用于处理鼠标悬停事件

- boolean onHover(View v, MotionEvent event);

##### AttachInfo
###### InvalidateInfo
用于post一个 invalidate(int, int, int, int) messages 给Handler。

```java 
76 
77     public static class More ...SimplePool<T> implements Pool<T> {
78         private final Object[] mPool;
79 
80         private int mPoolSize;

        
Creates a new instance.
Parameters:
maxPoolSize The max pool size.
Throws:
java.lang.IllegalArgumentException If the max pool size is less than zero.
88 
89         public More ...SimplePool(int maxPoolSize) {
90             if (maxPoolSize <= 0) {
91                 throw new IllegalArgumentException("The max pool size must be > 0");
92             }
93             mPool = new Object[maxPoolSize];
94         }
95 
96         @Override
97         @SuppressWarnings("unchecked")
98         public T More ...acquire() {
99             if (mPoolSize > 0) {
100                final int lastPooledIndex = mPoolSize - 1;
101                T instance = (T) mPool[lastPooledIndex];
102                mPool[lastPooledIndex] = null;
103                mPoolSize--;
104                return instance;
105            }
106            return null;
107        }
108
109        @Override
110        public boolean More ...release(T instance) {
111            if (isInPool(instance)) {
112                throw new IllegalStateException("Already in the pool!");
113            }
114            if (mPoolSize < mPool.length) {
115                mPool[mPoolSize] = instance;
116                mPoolSize++;
117                return true;
118            }
119            return false;
120        }
121
122        private boolean More ...isInPool(T instance) {
123            for (int i = 0; i < mPoolSize; i++) {
124                if (mPool[i] == instance) {
125                    return true;
126                }
127            }
128            return false;
129        }
130    }
```

### 构造函数

- View(Context context)

简单通过context获取到resource，简单初始化了两个Flags，给mTouchSlop赋值，设置了滚动模式，初始化了mRenderNode,并根据targetSdkVersion对一些兼容性进行初始化,4个兼容性。
`context.getApplicationInfo().targetSdkVersion`

- View(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes)

- setImportantForAccessibility()这个是有关于辅助功能的
- notifySubtreeAccessibilityStateChangedIfNeeded()
- notifyViewAccessibilityStateChangedIfNeeded()
- onClick属性是使用反射机制实现的

### 一些方法
#### performClick()
该方法会触发三个比较重要的方法，一个是playSoundEffect(),一个是onClick(),一个是sendAccessibilityEvent()。其中第一个跟AttachInfo有关，第二个跟ListenerInfo有关，第三个跟
AccessibilityDelegate有关。
#### callOnClick()
跟performClick()不同，只会触发onClick()
#### performLongClick()
会调用`sendAccessibilityEvent()`，当该view没有消费`onLongClick()`方法时，及返回false，就会调用`showContextMenu()`方法，否则调用`performHapticFeedback()`。





