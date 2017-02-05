---
title: shouldInterceptTouchEvent
date: 2016-05-20 07:14:36
tags:
---

May 20, 2016 7:14 AM
接下来我们回头看`shouldInterceptTouchEvent()`方法。
当子View消费了事件的时候，就是无法使用`processTouchEvent()`方法了在OnTouchEvent里。除非你将`processTouchEvent()`方法放到其他地方执行。

```java
            case MotionEvent.ACTION_MOVE: {
                if (mInitialMotionX == null || mInitialMotionY == null) break;

                // First to cross a touch slop over a draggable view wins. Also report edge drags.
                final int pointerCount = MotionEventCompat.getPointerCount(ev);
                for (int i = 0; i < pointerCount; i++) {
                    final int pointerId = MotionEventCompat.getPointerId(ev, i);
                    final float x = MotionEventCompat.getX(ev, i);
                    final float y = MotionEventCompat.getY(ev, i);
                    final float dx = x - mInitialMotionX[pointerId];
                    final float dy = y - mInitialMotionY[pointerId];

                    final View toCapture = findTopChildUnder((int) x, (int) y);
重点1                    final boolean pastSlop = toCapture != null && checkTouchSlop(toCapture, dx, dy);
                    if (pastSlop) {
                        // check the callback's
                        // getView[Horizontal|Vertical]DragRange methods to know
                        // if you can move at all along an axis, then see if it
                        // would clamp to the same value. If you can't move at
                        // all in every dimension with a nonzero range, bail.
                        final int oldLeft = toCapture.getLeft();
                        final int targetLeft = oldLeft + (int) dx;
                        final int newLeft = mCallback.clampViewPositionHorizontal(toCapture,
                                targetLeft, (int) dx);
                        final int oldTop = toCapture.getTop();
                        final int targetTop = oldTop + (int) dy;
                        final int newTop = mCallback.clampViewPositionVertical(toCapture, targetTop,
                                (int) dy);
                        final int horizontalDragRange = mCallback.getViewHorizontalDragRange(
                                toCapture);
                        final int verticalDragRange = mCallback.getViewVerticalDragRange(toCapture);
                        if ((horizontalDragRange == 0 || horizontalDragRange > 0
                                && newLeft == oldLeft) && (verticalDragRange == 0
                                || verticalDragRange > 0 && newTop == oldTop)) {
                            break;
                        }
                    }
                    reportNewEdgeDrags(dx, dy, pointerId);
                    if (mDragState == STATE_DRAGGING) {
                        // Callback might have started an edge drag
                        break;
                    }

                    if (pastSlop && tryCaptureViewForDrag(toCapture, pointerId)) {
                        break;
                    }
                }
                saveLastMotion(ev);
                break;
            }
```
感觉跟`processTouchEvent()`方法里的Move在没有被设置为拖拽状态的时候有点像。
我们看看标记为重点1的地方，`final boolean pastSlop = toCapture != null && checkTouchSlop(toCapture, dx, dy);` 这里重点的代码为`checkTouchSlop(toCapture, dx, dy)`，跟在`processTouchEvent()`里是一样的，为了方便查看，再次上代码：
```java
    private boolean checkTouchSlop(View child, float dx, float dy) {
        if (child == null) {
            return false;
        }
        final boolean checkHorizontal = mCallback.getViewHorizontalDragRange(child) > 0;
        final boolean checkVertical = mCallback.getViewVerticalDragRange(child) > 0;

        if (checkHorizontal && checkVertical) {
            return dx * dx + dy * dy > mTouchSlop * mTouchSlop;
        } else if (checkHorizontal) {
            return Math.abs(dx) > mTouchSlop;
        } else if (checkVertical) {
            return Math.abs(dy) > mTouchSlop;
        }
        return false;
    }
```
其中主要是使用`Callback`里的的`getViewVerticalDragRange()`和`getViewHorizontalDragRange()`方法，只要其返回值大于0就行，默认的情况下是返回0的，所以当子View消费事件时，如果你也想让其可拖拽，就必须在重写两个方法里，并将其设置大于0。
返回move里的代码`if ((horizontalDragRange == 0 || horizontalDragRange > 0
                                && newLeft == oldLeft) && (verticalDragRange == 0
                                || verticalDragRange > 0 && newTop == oldTop)) {
                            break;
                        }`
这里说明了只要有一个手指无法使其移动，那么其他手指也是一样的，没必要再尝试其他手指使其移动，这是无用功，因为不可能出现食指可以使其移动，而中指无法使其移动的这种情况。
**当然还有注意里面的break，也就是说只要出现一根手指能移动的情况，就不需要再判断其他的手指了（在每次move中）**
**shouldInterceptTouchEvent中的move少了processTouchEvent事件中的dragTo()，也就是说不需要在move一开始时进行` if (mDragState == STATE_DRAGGING)`这样的判断，因为拖拽是交给processTouchEvent的**

**还有关于`getViewVerticalDragRange()`和`getViewHorizontalDragRange()`方法，如果你想在第一次点击时没有点到可拖拽的view，但是想在移动过程中如果触碰到可以拖拽的view，这时你想让其跟着手指移动，那么这两个方法必须返回大于0.如果你不想在手指移动过程中触碰到的view跟着手指移动，那么就必须为0。下面为processTouchEvent的move：**

```java
            case MotionEvent.ACTION_MOVE: {
                if (mDragState == STATE_DRAGGING) {
                    final int index = MotionEventCompat.findPointerIndex(ev, mActivePointerId);
                    final float x = MotionEventCompat.getX(ev, index);
                    final float y = MotionEventCompat.getY(ev, index);
                    final int idx = (int) (x - mLastMotionX[mActivePointerId]);
                    final int idy = (int) (y - mLastMotionY[mActivePointerId]);

                    dragTo(mCapturedView.getLeft() + idx, mCapturedView.getTop() + idy, idx, idy);

                    saveLastMotion(ev);
                } else {
                    // Check to see if any pointer is now over a draggable view.
                    final int pointerCount = MotionEventCompat.getPointerCount(ev);
                    for (int i = 0; i < pointerCount; i++) {
                        final int pointerId = MotionEventCompat.getPointerId(ev, i);
                        final float x = MotionEventCompat.getX(ev, i);
                        final float y = MotionEventCompat.getY(ev, i);
                        final float dx = x - mInitialMotionX[pointerId];
                        final float dy = y - mInitialMotionY[pointerId];

                        reportNewEdgeDrags(dx, dy, pointerId);
                        if (mDragState == STATE_DRAGGING) {
                            // Callback might have started an edge drag.
                            break;
                        }

                        final View toCapture = findTopChildUnder((int) x, (int) y);
                        if (checkTouchSlop(toCapture, dx, dy) &&
                                tryCaptureViewForDrag(toCapture, pointerId)) {
                            break;
                        }
                    }
                    saveLastMotion(ev);
                }
                break;
            }
```

挑出证明上述观点的代码：

```java
final View toCapture = findTopChildUnder((int) x, (int) y);
                        if (checkTouchSlop(toCapture, dx, dy) &&
                                tryCaptureViewForDrag(toCapture, pointerId)) {
                            break;
                        }
```
**因为`checkTouchSlop()`方法只有`getViewVerticalDragRange()`和`getViewHorizontalDragRange()`大于0时才有可能返回true。**

`shouldInterceptTouchEvent()`与`processTouchEvent()`不同之处在于没有`dragTo()`和多了以下代码:

```java
                    if (pastSlop) {
                        // check the callback's
                        // getView[Horizontal|Vertical]DragRange methods to know
                        // if you can move at all along an axis, then see if it
                        // would clamp to the same value. If you can't move at
                        // all in every dimension with a nonzero range, bail.
                        final int oldLeft = toCapture.getLeft();
                        final int targetLeft = oldLeft + (int) dx;
                        final int newLeft = mCallback.clampViewPositionHorizontal(toCapture,
                                targetLeft, (int) dx);
                        final int oldTop = toCapture.getTop();
                        final int targetTop = oldTop + (int) dy;
                        final int newTop = mCallback.clampViewPositionVertical(toCapture, targetTop,
                                (int) dy);
                        final int horizontalDragRange = mCallback.getViewHorizontalDragRange(
                                toCapture);
                        final int verticalDragRange = mCallback.getViewVerticalDragRange(toCapture);
                        if ((horizontalDragRange == 0 || horizontalDragRange > 0
                                && newLeft == oldLeft) && (verticalDragRange == 0
                                || verticalDragRange > 0 && newTop == oldTop)) {
                            break;
                        }
                    }
```

因为当down被子view消费了，如果点击的view是无法移动的，或者移动不够，那么就没必要再判断下去了，可以break了，也就是说没必要进行reportNewEdgeDrags(dx, dy, pointerId)和tryCaptureViewForDrag(toCapture, pointerId)，因为view根本无法移动。可以让view继续消费move事件。但是processTouchEvent里就不同，因为没有子view消费事件，reportNewEdgeDrags(dx, dy, pointerId)还是有必要做的，说不定是该操作就是为了拉出侧边栏或者移动在移动手指过程中触碰到的view。

