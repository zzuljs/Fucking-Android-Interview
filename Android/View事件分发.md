# 1 View事件分发机制
Android的View事件分发机制是一个责任链模式的过程，用于确定和处理触摸事件  

三个角色：  

1. Activity：负责分发`dispatchTouchEvent`和消费`onTouchEvent`两个方法，事件由`ViewRootImpl`中的`DecorView.dispatchTouchEvent`分发  

2. ViewGroup：拥有分发、拦截和消费三个方法，对于一个根ViewGroup而言，点击事件产生后，首先传递给它，`dispatchTouchEvent`会被调用，此时ViewGroup可以选择`onInterceptTouchEvent = true`进行拦截，那么事件不会往下传递，直接回调`onTouchEvent`进行消费，否则就先往下传递给子View，子View的`dispatchTouchEvent`调用，重复这个过程  

3. View：只有分发和消费两个方法，`onTouchEvent`方法返回值为`true`表示消费事件，那么这个事件到这里就终止了，否则回传父View的`onTouchEvent`方法  

# 2 View的onTouchEvent、OnClickListener.onClick和OnTouchListener.onTouch方法，三者的优先级  

`dispatchTouchEvent`是这三个方法的入口，`OnTouchListener.onTouch`优先级最高，如果返回`false`，`onTouchEvent`会继续触发，返回`true`表示消费掉事件，`onTouchEvent`以及`OnClickListener.onClick`都不会触发  

`onClick`在`onTouchEvent`中执行，如果`onTouchEvent`在`event == ACTION_DOWN`时返回`true`，那么后续`ACTION`不会继续分发到`onTouchEvent`中，`onClick`也无法被回调

```java
// View.java
public boolean dispatchTouchEvent(MotionEvent event) {
    // ... 简化代码逻辑
    boolean result = false;

    // 1. ​**优先调用 OnTouchListener.onTouch()**
    if (mOnTouchListener != null 
            && (mViewFlags & ENABLED_MASK) == ENABLED 
            && mOnTouchListener.onTouch(this, event)) {
        result = true; // 事件被消费
    }

    // 2. 若未被消费，调用 onTouchEvent()
    if (!result && onTouchEvent(event)) {
        result = true;
    }

    return result;
}


// View.java
public boolean onTouchEvent(MotionEvent event) {
    // ... 简化代码逻辑
    
    final int action = event.getAction();
    if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
        switch (action) {
            case MotionEvent.ACTION_UP:
                // ... 处理抬起事件
                if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                    // ​**触发 OnClickListener**
                    performClickInternal(); 
                }
                break;
            case MotionEvent.ACTION_DOWN:
                // ... 处理按下事件（如长按检测）
                break;
        }
        return true; // 消费事件
    }
    return false; // 不消费事件
}
```

# 3 onTouch和onTouchEvent的区别  

`onTouch`是View的`OnTouchListener`接口中定义的方法，`onTouchEvent`是View内成员方法，通常自定义View时重写`onTouchEvent`方法来处理事件。当View绑定了`OnTouchListener`之后，touch事件触发时，`onTouch`方法会回调，优先级高于`onTouchEvent`，如果View没有绑定`OnTouchListener`，或`onTouch`返回`false`，系统会调用`onTouchEvent`处理事件  

`onTouch`适合外部监听，常用于临时拦截某个View的触摸行为，`onTouchEvent`适合自定义View，用于自定义交互事件  

# 4 ACTION_CANCEL什么时候触发  

1. 如果父View拦截了`ACTION_UP`或`ACTION_MOVE`，在第一次父View拦截消息瞬间，子View不再接受后续消息，然后收到`ACTION_CANCEL`事件  
2. 系统中断，比如正在触摸，被来电、弹窗等打断  
3. 手势冲突，比如`ViewPager`嵌套`RecyclerView`  

处理`ACTION_CANCEL`的目的是，当事件由于某种原因被取消时，原先的控件需要处理后续状态，比如按钮状态等

# 5 事件是先到DecorView还是先到Window  

先到`DecorView`，然后到`Window`，最后`Activity`兜底（如果存在`Activity`）  

`WindowInputEventReceiver.onInputEvent()` → 
   `ViewRootImpl.enqueueInputEvent()` → 
   `ViewRootImpl.processInputEvent()` → 
   `DecorView.dispatchPointerEvent()` → 
   `ViewGroup.dispatchTouchEvent()` → 
   `View.onTouchEvent()`（逐级回溯） → 
   `Window.superDispatchTouchEvent()`（二次分发） → 
   `Activity.dispatchTouchEvent()` → 
   `Activity.onTouchEvent()`  

这是一种责任链模式，从`View`->`Window`->`Activity`逐级传递，每层都可拦截  

# 6 点击事件被拦截，但是想传到下面的View，如何操作  

方法一：手动处理事件分发  

```java

// 需要传递事件的子View
private View targetChild;


// 父View中处理事件分发
@Override
public boolean onTouchEvent(MotionEvent event) {
    if (event.getAction() == MotionEvent.ACTION_DOWN) {
        // 找到目标子View
        targetChild = findChildUnder(event.getX(), event.getY());
    }
    if (targetChild != null) {
        // 转换坐标并分发事件
        MotionEvent childEvent = MotionEvent.obtain(event);
        childEvent.offsetLocation(-targetChild.getLeft(), -targetChild.getTop());
        boolean handled = targetChild.dispatchTouchEvent(childEvent);
        childEvent.recycle();
        return handled;
    }
    return super.onTouchEvent(event);
}
```

方法二：子View调用`requestDisallowInterceptTouchEvent`  

比如处理`ScrollView`和`RecyclerView`嵌套带来的滑动冲突的问题，假设父容器是`ScrollView`，垂直滑动，子View是`RecyclerView`，横向滑动  

```java
// 在 RecyclerView 的 onTouchEvent 中
@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            // 禁止父容器拦截后续事件
            getParent().requestDisallowInterceptTouchEvent(true);
            break;
        case MotionEvent.ACTION_MOVE:
            // 判断滑动方向
            if (isHorizontalScroll(event)) {
                // 处理横向滑动（RecyclerView 自身处理）
                return true;
            } else {
                // 如果是垂直滑动，允许父容器拦截
                getParent().requestDisallowInterceptTouchEvent(false);
            }
            break;
    }
    return super.onTouchEvent(event);
}
```
注意，这个方案需要父View放行`ACTION_DOWN`事件，父View如果无区分拦截所有事件，那子View是无法收到`onTouchEvent`回调的  

通常而言，父View不应该拦截`ACTION_DOWN`，因为事件分发是一组连续事件，`DOWN`是第一个事件，只要有其中一个事件被拦截，后续事件都将无法收到  

# 7 如何解决View的事件冲突  


方法一：外部拦截  

通过父View.onInterceptTouchEvent动态决定拦截哪些事件  
例如：父View与子View嵌套导致的滑动冲突  

```java
public class ParentView extends ViewGroup {
    private float mLastX, mLastY;

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        float x = ev.getX();
        float y = ev.getY();

        switch (ev.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                // 必须放行ACTION_DOWN，否则子View无法接收后续事件
                intercepted = false;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算滑动方向
                float dx = x - mLastX;
                float dy = y - mLastY;
                if (Math.abs(dx) > Math.abs(dy)) {
                    // 水平滑动，不拦截（交给子View处理）
                    intercepted = false;
                } else {
                    // 垂直滑动，父View拦截
                    intercepted = true;
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
        }

        mLastX = x;
        mLastY = y;
        return intercepted;
    }
}
```  

方法二：内部拦截  

子View通过requestDisallowInterceptTouchEvent主动请求父View不拦截事件  
这种方案需要父View放行ACTION_DOWN事件  

```java
public class ChildView extends View {
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 请求父View不拦截后续事件
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                // 处理子View的滑动逻辑
                if (isHorizontalScroll(event)) {
                    // 消费事件
                    return true;
                } else {
                    // 允许父View拦截垂直滑动
                    getParent().requestDisallowInterceptTouchEvent(false);
                }
                break;
        }
        return super.onTouchEvent(event);
    }
}
```

# 8 在ViewGroup中的onTouchEvent中消费ACTION_DOWN事件，ACTION_UP事件是怎么传递的  

一个事件序列只能被一个View拦截和消耗，因为一旦一个元素拦截了此事件，那么同一个事件序列内所有事件都会被该元素处理  

调用链：   
（DOWN事件开始传递）
Activity.dispatchTouchEvent->ViewGroup.dispatchTouchEvent->ViewGroup.onInterceptTouchEvent->View.dispatchTouchEvent->View.onTouchEvent->ViewGroup.onTouchEvent(此处DOWN被消费)->  
（UP事件开始传递）
Activity.dispatchTouchEvent->（DOWN事件被消费，不再过问是否拦截）ViewGroup.dispatchTouchEvent->ViewGroup.onTouchEvent  


# 9 Activity ViewGroup和View都不消费ACTION_DOWN，那么ACTION_UP事件是怎么传递的    

Activity.dispatchTouchEvent->ViewGroup.dispatchTouchEvent->ViewGroup.onInterpectTouchEvent->View.dispatchTouchEvent->View.onTouchEvent->ViewGroup.onTouchEvent->Activity.onTouchEvent  

# 10 同时对父View和子View设置点击方法，优先响应哪个  

默认情况下，子View优先响应点击事件  

从源码角度来看，父View先收到这个事件，如果onInterceptTouchEvent没有拦截的话（返回true表示拦截），通过dispachTouchEvent分发到子View  

子View重复这个过程，一直向下，如果子View消费了这次事件（注册点击监听、响应点击事件），那么子View的dispatchTouchEvent返回true，父View的onTouchEvent还有点击监听都不再响应  

如果子View不消费这个点击事件，那么父View的onTouchEvent会回调，然后处理事件  

# 11 requestDisallowInterceptTouchEvent的调用时机  

requestDisallowInterceptTouchEvent是处理父子View事件冲突的关键方法，通过设置标记位`FLAG_DISALLOW_INTERCEPT`，允许子View阻止父View禁用事件传递，从而获得事件传递控制权  

例如：
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_MOVE:
            if (子View需要获取事件) {
                // 继续禁止父 View 拦截
                getParent().requestDisallowInterceptTouchEvent(true);
            } else {
                // 允许父 View 拦截后续事件
                getParent().requestDisallowInterceptTouchEvent(false);
            }
            return true;
    }
    return super.onTouchEvent(event);
}
```
