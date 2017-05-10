---
layout: post

title: Instruments

date: 2017-03-14 15:34:24.000000000 +09:00

---


##  Analyze
+ 静态分析，
+ 可以检查可疑的内存泄露
+ 逻辑错误

## Timer Profiler
观察CPU的使用率，注意有没有异常出现的峰值情况。
观察函数执行的时间单位，可以查出程序变慢的原因，从函数栈中看见执行函数的花费的时间分布，来优化耗时部分的操作。
### 调试选项
+ Separate by Thread：按线程分开做分析，这样更容易揪出那些吃资源的问题线程。特别是对于主线程，它要处理和渲染所有的接口数据，一旦受到阻塞，程序必然卡顿或停止响应。
+ Invert Call Tree：反向输出调用树。把调用层级最深的方法显示在最上面，更容易找到最耗时的操作。
+ Hide Missing Symbols：隐藏缺失符号。如果 dSYM 文件或其他系统架构缺失，列表中会出现很多奇怪的十六进制的数值，用此选项把这些干扰元素屏蔽掉，让列表回归清爽。
+ Hide System Libraries：隐藏系统库文件。过滤掉各种系统调用，只显示自己的代码调用。
+ Flattern Recursion：拼合递归。将同一递归函数产生的多条堆栈（因为递归函数会调用自己）合并为一条。
+ Top Functions：找到最耗时的函数或方法。

## Leaks
下面两种都能清晰的看见内存泄露的对象
![](/assets/images/WX20170507-134808@2x.png)
![](/assets/images/WX20170507-134307@2x.png)

+ [Instruments-Leaks工具的归纳总结](http://www.cocoachina.com/ios/20161013/17759.html)
## Core Animation

### 调试选项
+ Color Blended Layers 
	+ 监测某些区域的图层混合耗费GPU资源比较严重
	+ UI设计师提供关闭alpha通道的图片
	+ opaque = true && alpha = 1.0
	+ 给backgroundColor设置默认值
+ ColorHitsGreenandMissesRed
	+ 当开启shouldRasterizep时，开启了离屏渲染，并且会缓存图层的绘制,这个选项说明了命中缓存的区域
+ Color Copied Images
	+ 监测GPU需要借助CPU转化不能识别的颜色格式，重新生成图片资源到缓存区
	+ UI设计师提供可以查看图片颜色有哪些不同
+ Color Misaligned Images
	+ 监测像素是否对其，通常是因为图片不正常缩放引起的
	+ 选用合适尺寸的图片，放大倍率。
+ Color Offscreen-Rendered Yellow
	+ 监测哪些区域触发了离屏渲染
	+ 用 shadowPath 优化 shadow
	+ 利用 绘图缓存 shouldRasterize = true
+ Color Compositing Fast-Path Blue
	+ 对任何直接使用OpenGL绘制的图层进行高亮 
+ Flash updated Regions
	+ 监测Core Graphics绘制的图层时 频繁重绘的区域。 
	+ 检查优化自己的Core Graphics代码。

### 离屏渲染什么时候被触发？

+ shouldRasterize（光栅化）
+ masks（遮罩）
+ shadows（阴影）
+ edge antialiasing（抗锯齿）
+ group opacity（不透明）
+ 复杂形状设置圆角等
+ 渐变

### 优化或者避免离屏渲染

+ 处理阴影策略
	1. 给shadow赋值一个贝塞尔表述shadow路径，就可以避免每次刷新都要重新计算shadow的离屏渲染。
	2. view.layer.shadowPath = UIBezierPath(rect: view.bounds)
+ 处理圆角策略
	+ UIImageView 
		1. 用圆角蒙版Image的UIImageView，addSubView覆盖到头像上
		2. 设计师直接做好带圆角效果的图
		3. 用 Core Graphics 在后台画一张图给 self.image ,可以用贝塞尔曲线。
	+ UIView 
		1. cornerRadius 只修改border和backgroundColor 不会修剪图片
		2. cornerRadius > 0 && masksToBounds = true  才会触发离屏渲染
		3. 设置shouldRasterize = true 可以为mask，Corner，shadow产生的离屏渲染优化缓存。
		4.  view.layer.shouldRasterize = true && view.layer.rasterizationScale = view.layer.contentsScale
	+ UILabel 文本类要设置
		+ label.layer.backgroundColor = aColor
		+ label.layer.cornerRadius = 5
+ 获取图片策略
	+ imageNamed方法获取的图片会缓存在内存中，适合滑动cell里的图片频繁多次使用。
	+ imageWithContentsOfFile 没有缓存，用完直接释放，适合使用次数少的图。

+ [UIKit性能调优实战讲解](http://www.jianshu.com/p/619cf14640f3)
+ [iOS核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/chapter12/instruments.html)





