title: View的事件分发机制
date: 2015-03-25 19:18:15
categories: Android
---

View事件的传递过程是dispatchTouchEvent  ->  mOnTouchListener.onTouch -> onTouchEvent -> onClick

<!--more-->

先来看dispatchTouchEvent 方法中


 ````
public boolean dispatchTouchEvent(MotionEvent event) {
        if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&
                mOnTouchListener.onTouch(this, event)) {
            return true;
        }
        return onTouchEvent(event);
    }

````

第2、3行先判断mOnTouchListener是否为null和这个view是否显示。


如果不为空且显示，则就会执行mOnTouchListener.onTouch(this, event)这个方法，onTouch方法如果返回true，则这个会被消费掉，否则执行onTOuchEvent方法，所以onTouch方法和onTouchEvent方法是可以同时存在的，也就是说当onTouch方法返回false，就可以往下执行onTouchEvent了。


假设onTouch方法中返回了false，那么就往下执行了onTouchEvent方法，这个方法有点长，一段一段的来看



````
if ((viewFlags & ENABLED_MASK) == DISABLED) {
    // A disabled view that is clickable still consumes the touch
    // events, it just doesn't respond to them.
    return (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
}

if (mTouchDelegate != null) {
    if (mTouchDelegate.onTouchEvent(event)) {
        return true;
    }
}
````

判断当前View为Disabled，而且可点击则会被消费掉(返回true)，有触摸委派，且它的onTouchEvent方法为true，事件也会被消费掉。


````
if (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;
                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {
                    // take focus if we don't have it already and we should in
                    // touch mode.
                    boolean focusTaken = false;
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                        focusTaken = requestFocus();
                    }

                    if (!mHasPerformedLongPress) {
                        // This is a tap, so remove the longpress check
                        removeLongPressCallback();

                        // Only perform take click actions if we were in the pressed state
                        if (!focusTaken) {
                            // Use a Runnable and post this rather than calling
                            // performClick directly. This lets other visual state
                            // of the view update before click actions start.
                            if (mPerformClick == null) {
                                mPerformClick = new PerformClick();
                            }
                            if (!post(mPerformClick)) {
                                performClick();
                            }
                        }
                    }

                    if (mUnsetPressedState == null) {
                        mUnsetPressedState = new UnsetPressedState();
                    }

                    if (prepressed) {
                        mPrivateFlags |= PRESSED;
                        refreshDrawableState();
                        postDelayed(mUnsetPressedState,
                                ViewConfiguration.getPressedStateDuration());
                    } else if (!post(mUnsetPressedState)) {
                        // If the post failed, unpress right now
                        mUnsetPressedState.run();
                    }
                    removeTapCallback();
                }
                break;

            case MotionEvent.ACTION_DOWN:
                if (mPendingCheckForTap == null) {
                    mPendingCheckForTap = new CheckForTap();
                }
                mPrivateFlags |= PREPRESSED;
                mHasPerformedLongPress = false;
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                break;

            case MotionEvent.ACTION_CANCEL:
                mPrivateFlags &= ~PRESSED;
                refreshDrawableState();
                removeTapCallback();
                break;

            case MotionEvent.ACTION_MOVE:
                final int x = (int) event.getX();
                final int y = (int) event.getY();

                // Be lenient about moving outside of buttons
                int slop = mTouchSlop;
                if ((x < 0 - slop) || (x >= getWidth() + slop) ||
                        (y < 0 - slop) || (y >= getHeight() + slop)) {
                    // Outside button
                    removeTapCallback();
                    if ((mPrivateFlags & PRESSED) != 0) {
                        // Remove any future long press/tap checks
                        removeLongPressCallback();

                        // Need to switch from pressed to not pressed
                        mPrivateFlags &= ~PRESSED;
                        refreshDrawableState();
                    }
                }
                break;
        }
        return true;
    }
````

当手指按下view的时候（第49行-56行），将mPrivateFlags设置为PREPRSSED，发送延时任务(postDelayed)，在115毫秒之后会将这个flag标记为长按事件.

当手指在view上抬起的时候(第4-47行)，判断mPrivateFlags是否为PREPRESSED。

如果为真，则会继续判断当前view是否可获得焦点，如果可以获得则执行requestFocus方法。

14-16行，view控件是否有长按事件，如果有则移除掉长按事件。

19-29行，前面执行了requestFocus方法，得到了fousTaken这个变量，判断可获得焦点进入if判断中，mPerformClick为null，创建一个preformClick实例，使用代码主动调用控件的点击事件，然后添加到执行队列中。

36-45行，将mPrivateFlags还原，刷新控件背景，并执行mUnsetPressedState。最后移除轻触检测。


最后来看move时的代码，64-84行，判断当前坐标位置是否还在控件上面，如果不在，就移除轻触检测，移除长按检测，将控件还原，刷新控件背景。



