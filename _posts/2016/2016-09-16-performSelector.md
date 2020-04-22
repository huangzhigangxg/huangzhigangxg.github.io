---
layout: post

title: performSelector的总结

date: 2016-09-15 15:32:24.000000000 +09:00

---

PerformSelector 可以用来线程间传递消息，原理就是将传进来的Selector变成Runloop线程的Custom Input Sources  自定义源,并在下一次或者将来的循环Next Loop中注册并激活。

这些API分成两部分：
     一部分记录在RunLoop.h里:
```
/**************** Delayed perform ******************/
extension NSObject {

    public func performSelector(aSelector: Selector, withObject anArgument: AnyObject?, afterDelay delay: NSTimeInterval, inModes modes: [String])
    public func performSelector(aSelector: Selector, withObject anArgument: AnyObject?, afterDelay delay: NSTimeInterval)
    public class func cancelPreviousPerformRequestsWithTarget(aTarget: AnyObject, selector aSelector: Selector, object anArgument: AnyObject?)
    public class func cancelPreviousPerformRequestsWithTarget(aTarget: AnyObject)
}
```
     另外一部分在NSThread.h里：

```
extension NSObject {

    public func performSelectorOnMainThread(aSelector: Selector, withObject arg: AnyObject?, waitUntilDone wait: Bool, modes array: [String]?)
    public func performSelectorOnMainThread(aSelector: Selector, withObject arg: AnyObject?, waitUntilDone wait: Bool)

    @available(iOS 2.0, *)
    public func performSelector(aSelector: Selector, onThread thr: NSThread, withObject arg: AnyObject?, waitUntilDone wait: Bool, modes array: [String]?)
    @available(iOS 2.0, *)
    public func performSelector(aSelector: Selector, onThread thr: NSThread, withObject arg: AnyObject?, waitUntilDone wait: Bool)

    @available(iOS 2.0, *)
    public func performSelectorInBackground(aSelector: Selector, withObject arg: AnyObject?)
}
```

第一部分在RunLoop里的方法主要作用是
     延时操作，是异步的但不会生产新的线程，其实就是在当前线程的Runloop添加 Timer Source时间源. 这里注意的是，如果在子线程调用这些方法，Selector可能是不会被执行，即使delay为0，因为子线程的RunLoop默认是关闭的。添加 NSRunLoop.currentRunLoop().runUntilDate(NSDate(timeIntervalSinceNow: 2))
就可以了。

第二部分在NSThread里的方法主要作用是
     线程间通信，其实就是在对方的RunLoop里添加Custom Input Source 自定义输入源 ，来调用这个Selector。这里注意的是 waitUntilDone 为YES时 会阻塞自己等待对方线程执行完，为NO时，就不会阻塞自己。如果都为同一个线程，当为YES，立刻执行阻塞下面的代码，为NO不会阻塞，则会放到下一次RunLoop里，找机会执行。