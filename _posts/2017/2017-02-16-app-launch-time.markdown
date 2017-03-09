---
layout: post

title: 测量启动时间

date: 2017-02-16 15:32:24.000000000 +09:00

---
要测量的时间分为main函数之前动态库的装载 到 CoreAnimation 渲染成功第一个帧。

## main函数之前

在 Xcode 中 Edit scheme -> Run -> Auguments 添加 `DYLD_PRINT_STATISTICS` 变量 设为1 ，可以打印动态连结库的载入消耗时间。

```
Total pre-main time:  89.36 milliseconds (100.0%) //main之前的总时间
         dylib loading time:  35.23 milliseconds (39.4%)
        rebase/binding time:  17.64 milliseconds (19.7%)
            ObjC setup time:  22.18 milliseconds (24.8%)
           initializer time:  14.18 milliseconds (15.8%)//导入非系统的dylib耗时
           slowest intializers : //非系统的dylib 最慢的几个
               libSystem.dylib :   2.01 milliseconds (2.2%)
   libBacktraceRecording.dylib :   3.18 milliseconds (3.5%)
                       RxCocoa :   3.29 milliseconds (3.6%)
```



## main函数之后

需要自己创建main.swift,并屏蔽掉//@UIApplicationmain . 在info.plist中加上main.storyboard. 在main.swift 中自己写main函数入口：

##### main.swift

{% highlight swift %}

let MainStartTime = CFAbsoluteTimeGetCurrent()

UIApplicationMain(CommandLine.argc,
                  UnsafeMutableRawPointer(CommandLine.unsafeArgv).bindMemory(to: UnsafeMutablePointer<Int8>.self, capacity: Int(CommandLine.argc)),
                  nil,

                  NSStringFromClass(AppDelegate.self))

{% endhighlight %}

##### FirstViewController.swift

{% highlight swift %}

    override func viewDidLoad() {

        super.viewDidLoad()

        print(“Lauched in \(CFAbsoluteTimeGetCurrent() - MainStartTime) s" )

    }

{% endhighlight %}

### 优化

-  ``+load()``在main 前就被调用 尽量延迟到initialize,并用 `gcd : dispatch_once`线程保护起来
- 纯代码加载首页 
- 启动时慎用 ``NSUserDefault``    `` NSLog``
- 减小启动网络接口
- ``coredata`` 转 ``fmdb`` 大概减小300ms
- 启动时首页加载缓
- 非UI相关业务 并行初始化 ``gcd barrier_sync ``初始化各种 sdk 等 大概减小100ms

### 参考

- [优化 App 的启动时间](http://yulingtianxia.com/blog/2016/10/30/Optimizing-App-Startup-Time/)
- [iOS 应用性能测试的相关方法、工具及技巧](http://www.10tiao.com/html/327/201605/2652154806/2.html)

