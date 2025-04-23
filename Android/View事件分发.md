# 1 View事件分发机制
Android的View事件分发机制是一个责任链模式的过程，用于确定和处理触摸事件  

1. 事件传递流程  
触摸事件从Activity开始，传递到Window，再到达ViewGroup（通常是DecorView）

# 2 View的onTouchEvent、OnClickListener.onClick和OnTouchListener.onTouch方法，三者的优先级  

dispatchTouchEvent是这个三个方法的入口，OnTouchListener.onTouch优先级最高，如果返回false，onTouchEvent会继续出发，返回true表示消费掉事件，onTouchEvent以及OnClickListener.onClick都不会触发；onClick在onTouchEvent中执行，如果onTouchEvent在event==ACTION_DOWN时返回true，那么后续ACTION不会继续分发到onTouchEvent中，onClick也无法被回调

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

onTouch是View的OnTouchListener接口中定义的方法，onTouchEvent是View内成员方法，通常自定义View时重写onTouchEvent方法来处理事件。当View绑定了OnTouchListener之后，touch事件触发时，onTouch方法会回调，优先级高于onTouchEvent，如果View没有绑定OnTouchListener，或onTouch返回false，系统会调用onTouchEvent处理事件  

onTouch适合外部监听，常用于临时拦截某个View的触摸行为，onTouchEvent适合自定义View，用于自定义交互事件  

# 4 ACTION_CANCEL什么时候触发  

1. 如果父View拦截了ACTION_UP或ACTION_MOVE，在第一次父View拦截消息瞬间，子View不再接受后续消息，然后收到ACTION_CANCEL事件  
2. 系统中断，比如正在触摸，被来电、弹窗等打断  
3. 手势冲突，比如ViewPager嵌套RecyclerView  

处理ACTION_CANCEL的目的是，当事件由于某种原因被取消时，原先的控件需要处理后续状态，比如按钮状态等

# 5 事件是先到DecorView还是先到Window  

先到DecorView，然后到Window，最后Activity兜底（如果存在Activity）  

WindowInputEventReceiver.onInputEvent() → 
   ViewRootImpl.enqueueInputEvent() → 
   ViewRootImpl.processInputEvent() → 
   DecorView.dispatchPointerEvent() → 
   ViewGroup.dispatchTouchEvent() → 
   View.onTouchEvent()（逐级回溯） → 
   Window.superDispatchTouchEvent()（二次分发） → 
   Activity.dispatchTouchEvent() → 
   Activity.onTouchEvent()  

这是一种责任链模式，从View->Window->Activity逐级传递，每层都可拦截  

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

比如处理ScrollView和RecyclerView嵌套带来的滑动冲突的问题，假设父容器是ScrollView，垂直滑动，子View是RecyclerView，横向滑动  

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
注意，这个方案需要父View放行ACTION_DOWN事件，父View如果无区分拦截所有事件，那子View是无法收到onTouchEvent回调的  

通常而言，父View不应该拦截ACTION_DOWN，因为事件分发是一组连续事件，DOWN是第一个事件，只要有其中一个事件被拦截，后续事件都将无法收到  

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

# 9 Activity ViewGroup和View都不消费ACTION_DOWN，那么ACTION_UP事件是怎么传递的    

# 10 同时对父View和子View设置点击方法，优先响应哪个  

默认情况下，子View优先响应点击事件，

# 11 requestDisallowInterceptTouchEvent的调用时机  