# 1 Handler的实现原理  
# 2 子线程中能不能直接new一个Handler，为什么主线程可以  
# 3 Handler导致的内存泄漏原因以及解决方案  
# 4 一个线程可以有几个Handler，几个Looper，几个MessageQueue对象  
# 5 Message对象创建方式有哪些，分别有哪些区别  
# 6 Handler有哪些发送消息的方法  
# 7 Handler的post与sendMessage区别和应用场景  
# 8 Handler postDelay后消息队列有什么变化，假设先postDelay 10s，再postDelay 1s，怎么处理这2条消息  
# 9 MessageQueue是什么数据结构  
# 10 Handler怎么做到一个线程对应一个Looper，如何保证自由一个MessageQueue ThreadLocal在Handler机制中的作用是什么  
# 11 HandlerThread是什么？原理？使用场景有哪些？
# 12 IdleHandler使用场景  
# 13 消息屏障，同步屏障机制是什么  
# 14 子线程能不能更新UI  
# 15 为什么Android系统不建议子线程访问UI
# 16 Android中为什么主线程不会因为Looper.loop死循环卡死  
# 17 Handler消息机制中，一个Looper是如何区分多个Handler的？当Activity有多个Handler的时候，怎么区分当前消息由哪个Handler处理？处理Message的时候如何区分哪个callback？  
# 18 Looper.quit/quitSafely的区别  
# 19 通过Handler如何实现线程切换  
# 20 Handler如何与Looper关联  
# 21 Looper如何与Thread关联  
# 22 Looper.loop源码  
# 23 MessageQueue的enqueueMessage方法如何进行线程同步  
# 24 MessageQueue的next方法内部原理  
# 25 子线程中是否可以用MainLooper去创建Handler，Looper和Handler是否一定处于一个线程？  
# 26 ANR与Handler的联系  