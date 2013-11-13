---
layout: post
title: "当View点击时"
date: 2013-09-18 20:17
comments: true
categories: Android
---
微博上@弹棉花的孩子 这位同学，发表了如下微博:

![](/media/2013-09-18-dispatch-set-pressed/weibo.png)

那么在我印象里，给`View`设置`selector`，并且没有设置`onClickListener`时，父容器的点击会触发`selector`效果，我利用这个特性还做过不少效果，因此我认定，po主说的这个情况与MIUI无关。
为了证明MIUI是清白的，我决定简单的分析以下`View`点击的流程。

<!-- more -->

相关的代码肯定与`View`,`ViewGroup`的触摸事件相关，那么我们首先从`View`的`onTouchEvent`入手：
``` java
/**
     * Implement this method to handle touch screen motion events.
     *
     * @param event The motion event.
     * @return True if the event was handled, false otherwise.
     */
    public boolean onTouchEvent(MotionEvent event) {
    ...
                case MotionEvent.ACTION_DOWN:
                    mHasPerformedLongPress = false;

                    if (performButtonActionOnTouchDown(event)) {
                        break;
                    }

                    // Walk up the hierarchy to determine if we're inside a scrolling container.
                    boolean isInScrollingContainer = isInScrollingContainer();

                    // For views inside a scrolling container, delay the pressed feedback for
                    // a short period in case this is a scroll.
                    if (isInScrollingContainer) {
                        mPrivateFlags |= PFLAG_PREPRESSED;
                        if (mPendingCheckForTap == null) {
                            mPendingCheckForTap = new CheckForTap();
                        }
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                    } else {
                        // Not inside a scrolling container, so show the feedback right away
                        setPressed(true);
                        checkForLongClick(0);
                    }
                    break;
...
```


我们注意到，当`ACTION_DOWN`发生时，`setPressed()`会被调用：
``` java 
    /**
     * Sets the pressed state for this view.
     *
     * @see #isClickable()
     * @see #setClickable(boolean)
     *
     * @param pressed Pass true to set the View's internal state to "pressed", or false to reverts
     *        the View's internal state from a previously set "pressed" state.
     */
    public void setPressed(boolean pressed) {
        final boolean needsRefresh = pressed != ((mPrivateFlags & PFLAG_PRESSED) == PFLAG_PRESSED);

        if (pressed) {
            mPrivateFlags |= PFLAG_PRESSED;
        } else {
            mPrivateFlags &= ~PFLAG_PRESSED;
        }

        if (needsRefresh) {
            refreshDrawableState();
        }
        dispatchSetPressed(pressed);
    }
```

在`setPressed()`方法中，`refreshDrawableState()`会被调用，这个方法的作用是刷新`View`的`selector`状态，也就是显示出被点击的Drawable State，同时我们注意到，方法的最末尾调用了`dispatchSetPressed()`，从命名可以看出这个方法会对pressed状态做分发。

``` java 
    /**
     * Dispatch setPressed to all of this View's children.
     *
     * @see #setPressed(boolean)
     *
     * @param pressed The new pressed state
     */
    protected void dispatchSetPressed(boolean pressed) {
    }
```

在`View`中该方法没有具体实现，但注释已经说明此时此刻，答案离我们很近了。

以下是`ViewGroup`对该方法的实现：
``` java 
    @Override
    protected void dispatchSetPressed(boolean pressed) {
        final View[] children = mChildren;
        final int count = mChildrenCount;
        for (int i = 0; i < count; i++) {
            final View child = children[i];
            // Children that are clickable on their own should not
            // show a pressed state when their parent view does.
            // Clearing a pressed state always propagates.
            if (!pressed || (!child.isClickable() && !child.isLongClickable())) {
                child.setPressed(pressed);
            }
        }
    }
```


So, the answer is clear. 总的说来就是在点击了父容器之后，父容器会遍历每一个子View并设置其点击状态。

如何避免？
通过代码可以判断，如果子`View`本身是可以点击的，将不会受到父容器的pressedState影响，所以代码可以这么写
``` java
child.setClickable(true);
```
当然在xml中设置也是可以的。



