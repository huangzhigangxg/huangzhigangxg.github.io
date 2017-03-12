
---
layout: post

title: iDev开发者大会

date: 2016-11-12 15:32:24.000000000 +09:00

---

# iDev开发者大会

记录大会上的收获有些关键词涉及到的技术暂且留在这里，需要今后扩展和修正


## Swift开发服务器


TODO： kitura！了解 nginx？apache？的概念 做一个服务器demo！ 
DETAIL：客户端在交互与展示，服务器的重点在日志库进程检测库！


## 大项目组件化


DETAIL：大项目的业务线众多，必须用组件化方式隔离各个模块分别开发，开发要点：

+ 1 是有严格具体的组件化开发规范，
+ 2 组件之间的通信借助 `组件管理器 (中间件)`并且抽象出协议族，
+ 3 各个组件的跳转则需要有一个 `路由器（映射文件）`，支持网络更新，方便线上处理问题。
+ 4 代码组织方式 依靠 cocopods 的 PodFile
+ 5 注意的是高度的抽象 ，不能有模棱两可的组件，必须清晰，多个组件公用的东西 比如 数据模型 尽量下沉到静态库。
+ 6 通过WWDC视频 练练口语拼读吧。


## MacOX操作系统

DETAIL：带着问题去了解MacOX操作系统，

+ 系统调试器是什么原理呢？
MacOX linuc unix系统都有用户空间和内核空间，用户空间将操作进程交给内核空间将运算，内核空间的task运算完之后将结果返回给用户空间的进程，用户空间的进程pthread和内核空间的任务块task是一一对相应的。调试器查看的信息也就是这些内核空间里的cup寄存器数据以及内存里堆栈内容，调试器通过task_for_pid 获取被检测进程的id 然后调用系统内核内存管理的接口查看进程内存情况！而Linux 和 Unix 的pthread 本身就带有信息 可以访问！
+ 在MacOX操作系统为什么不能自己写调试器？ 
外部进程要 Entitlement 加上一个标签说明 ，说明想要访问其他进程的数据，才能从 threah 转task 才能继续查看内核中的 进程数据！苹果在审核时不会通过！
+ 切换进程的原理 ？
只是保留寄存器里的一些数据 以及当前的命令地址在哪，大约4K的数据，将这4k数据先保存到其他的地方，等切换回来时，重新恢复这4k数据。


## linker 和 loader


DETAIL:了解代码是如何编译载入内存并执行的

+ 1.file.h文件只是帮助编译期间识别类型索引等，编译之后就不会有file.h的数据了
+ 2.file.m文件经过clang命令 编译成 file.o  最后多个otherFiles.o文件都通过linker和loader连接在一起形成可执行文件
+ 3.静态库 就是.o的zip包 用来和并编译代码 之后可能会 lipo 一些其他.o文件
+ 4.动态库 因为不能把UIKit NSFoundation等等框架都合并到所有App里，所以要动态链接，好处是可以及时修改，还可以多个app共用UIKit NSFoundation等Cocoa框架
+ 5.连接器的原理呢？
file.m 文件经过编译器编译变成了file.o 当所有的otherFile.o 都链接在一起时就变成了可执行程序。那么他们是怎么链接到一起的呢？
分析下file.m文件里面 基本上都是实现的函数接口以及用到的其他文件中的函数的头文件引入，说白了就是函数的声明和实现，当遇见函数名时就将他符号化放在那，等合并到其他的otherFile.o 时，就会碰到这个函数名的实现。再将这个函数的地址填入到刚才空出来的位置上，这样多个otherFile.o 的函数名的符号化就合并完成，也就完成了静态链接。
那么动态链接的原理呢？
有些调用到的其他函数，在编译器没有实现的，先用特殊的声明方式，先空在那里并用Data段的一个结构体指向这个地址，等程序载入执行载入到内存时，系统将这个执行程序所需要的动态库依次加载进来。碰到这个函数名，就通过这个Data的结构体找到之前空出来的位置添加一个jmp address 将真正的函数地址传进去。这样动态链接库的函数名的符号化就合并完成，也就完成了动态链接。动态绑定也分为Non-Lazy Pointer Binding （程序启动，加载时绑定） 和 Lazy Pointer Binding （符号第一次被用到时绑定）iOSSDK绝大部份都是Lazy Binding！

+ 6.dyld在main函数前做了什么
  + 加载dylibs(LC_LOAD_DYLIB)程序指定的链接库
  + Rebase修正随机base地址偏移
  + Binding 确定 Non-Lazy Pointer 地址
  + Load Objective-c Runtime 加载所有类
  + Call Initializers：+load __attribute_(constructor)一些类的 load（） //不推荐用！延迟加载速度用initstalne方法 替换类方法 可以避免 加载速度慢 但是有可能重复？
main（）


+ 7.有些category 的静态库找不到方法因为 连接器发现没用用到这个库，这个库就不会被链接，目的是优化最后目标的大小 要添加 -objc 强行链接这个.o

+ 8.苹果是找到私有api的呢？
  + 查看 commands lc_load_dylib 是有否私有dylib otool -L
  + 查看 symbol table  有没有私有符号？ nm -u
  + 在 __TEXT, __objc_methname 代码段找objc selector的使用，找有没有 匹配的字符？
  + 在 __TEXT, __cstring 常量段中查找字符串的使用 
  + 可以将私有函数的名字分成几段当真正使用时在进行拼接，达到动态调用的目的

+ 9.Embed framework 
在IOS8之后苹果终于开始允许开发者有条件的使用动态链接库，这就是Embed framework，他解决的两个问题
就是使主app和extension之间的代码共享，和Swift的标准库不在系统中的而必须以这种方式加在每一个app bundle中。
注意 将不常用的Embed Framework 手工加载而不在main之加载用 会影响加载速度

+ 10.fishbook 替代系统的c函数
利用 动态链接 lazy 的特性自己重写 指针地址在data段 可重新改写函数地址
xcode 默认 strip 后回干掉内存里的符号表保护已经写好的 c函数

+ 11.运行中 怎么找到动态链接库！
```
NSString *bundlePath = [[NSBundle mainBundle]
pathForResource:@"DynamicPlugin" ofType:@"framework"];
NSBundle *bundle = [NSBundle bundleWithPath:bundlePath];
[bundle load];
Class cls = NSClassFromString(@"Sark");
```

+ 12.多利用动态链接库 提高编译速度 动态编译库是编译完的！可以把多个动态库打一个静态库！避免重复编译！
+ [参考](https://my.oschina.net/kaqijiang/blog/649632)


## 单元测试和自动化
DETAIL:必须将mvc改为mvvm！

+ 测试必须从项目刚开始时就开始写 边写开发代码一边写测试代码！才会越来越健壮！
+ 有重点问题频繁bug 可以写个测试重现bug 并且每次也会 重新验证
+ 尽量一份测试代码 测试多个模块 提高测试效率
+ xcode7 可以看测试代码的覆盖率 

TODO：jenkins远程自动化测试？ github PR ？

## 响应式编程

DETAIL：一般有callback 就算是响应式的编程

+ 响应式实现方式分为pull driver 和 push driver
+ Push driver的响应式 实现方式就是定义一个订阅者数组observers，当数值改变时setNew是对每一个订阅者推送结果！
+ RxSwift 是 Push driver！因为用户产生少数变化 导致数据和UI状态 生产更多种变化 一种由少向多的Push合适！ 当由多向少时就是Pull合适！
+ Pull driver的响应式 实现方式就是定义一个获取值得block，在获取数据是getValue方法时调用这个block，这个block就会一层一层 找到下面的具体的一些变化 组合起来得到最后的变化。

命令式编程：通过命令产生状态 然后判断 然后产生新的命令 为了解决计算问题
遇到异步就傻逼了 真正的UI场景都是异步问题



## 会后资料

iDev 苹果开发者大会视频直播地址：
http://m.quzhiboapp.com/?liveId=180

11.5照片地址：
http://vphotos.cn/vphotosgallery/index.html?vphotowechatid=5D7EF640B66B6BEB08B3CFFB14C2E6C3&from=timeline&isappinstalled=0

11.6照片地址：
http://vphotos.cn/vphotosgallery/index.html?vphotowechatid=7EDD524570CB0A1FF0AACE37525CE8DF&code=031qZ9nA13vSIg0yNgoA1OUbnA1qZ9nP&state=STATE

PPT下载地址：
https://github.com/devlinkcn/ppts_for_idev2016