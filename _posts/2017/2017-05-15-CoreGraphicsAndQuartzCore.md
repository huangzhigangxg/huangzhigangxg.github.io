---
layout: post

title: CoreGraphics And QuartzCore

date: 2017-05-15 15:34:24.000000000 +09:00

---


## 先了解框架
+ 他们都属于 Media Layer 层,用户绘制界面，这些框架如何分工呢？
+ UIKit.framework : 提供OC接口来使用已经封装好的经典控件，以及使他们支持交互事件功能
+ QuartzCore.framework
	+ CoreAnimation.h  : iOS的QuartzCore.framework只包含了CoreAnimation一个头文件，相对MacOS还包含CoreImage和CoreVideo，应该算是精简版，但不能说明QuartzCore只能做动画相关的事情，它提供了图形处理和视频图像处理的能力。简单来说就是：QuartzCore Framework负责把图形图像最终显示到屏幕上。CALayer就是QuartzCore的，而他要显示的数据模型来自CoreGraphics。

+ CoreGraphics.framework : 基于C函数的高级绘画引擎，处理绘画工作CGPath，变形操作CGAffineTransform，颜色管理CGColor，离屏渲染CGBitmapContextCreateImage，渲染模式patterns，渐变gradients，阴影效果，图形数据的创建和管理，PDF文档的创建和管理。

+ QuartzCore和CoreGraphics 区别是：CoreGraphics负责创建显示到屏幕上的数据模型，QuartzCore(CoreAnimation –> OpenGLES)负责把CoreGraphics创建的数据模型真正显示到屏幕上。

## 图形结构

CALayer像UIView可以addSubviews一样，都会有一个树型的存贮结构，叫 Layer Tree 图层树

+ UIView 视图树
+ CALayer 图层树
	+ 逻辑树(目标帧) ：逻辑树上的数据就是程序员用代码控制，可以set和get到的数据，CALayer – modelLayer()
	+ 动画树(当前帧) ：动画树是一个中间层，系统通过逻辑树上数据的改变，在动画树上从最初始值逐步修改到最终的值。CALayer – presentationLayer()
	+ 渲染树(下一帧) ：图层和动画打包提交到渲染服务后反序列化所得树，也就是已经提交到渲染服务的数据

## 渲染流程


1. Source0收到VSync，App就会利用CPU处理一次绘制更新数据然后提交到GPU去进行变换合成渲染，最后把结果放到帧缓冲区。也就是说系统以1/60s的频率发送VSync信号给App，去驱动界面的更新。
2. Source1会收到触摸事件，也会驱动App更新整个图层树 CALayer Tree，利用CATransaction提交到中间状态，等待CoreAnimation的BeforWaitingObserver触发时将这些中间状态打包给GPU。
3. 在BeforWaitingObserver触发时，会遍历所有收集好的需要更新的 CALayer，对他们发送是否重新布局 `[UIView layoutSubviews]`，或者重新绘制`[UIView drawRect];`的请求。
4. 通过这两个方法会修改CALayer的数据模型，之后将结果转换位图 Bitmap，通过OpenGLES把位图信息传送到GPU的缓冲区。之后将这些数据更新到渲染树上。

```
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()
    QuartzCore:CA::Transaction::observer_callback:
        CA::Transaction::commit();
            CA::Context::commit_transaction();
                CA::Layer::layout_and_display_if_needed();
                    CA::Layer::layout_if_needed();
                          [CALayer layoutSublayers];
                          [UIView layoutSubviews];
                    CA::Layer::display_if_needed();
                          [CALayer display];
                          [UIView drawRect];
//UIView作为CALayer的Delegate ，响应drawRect方法
```



## Core Graphics的绘制是如何修改CALayer的内容？
答案就是CGContext上下文，它定义了我们需要绘制的地方. 当我实现 CALayer 的 `-drawInContext:`方法, 会传进来当前的CGContext，还有在 UIView 的 drawrect 中通过 UIGraphicsGetCurrentContext() 也可以获得的CGContext。绘制到这个上下文中的内容将会被绘制到图层的备份区(图层的缓冲区)。

UIKit 方法总是绘制到最顶层的上下文中。你可以使用 UIGraphicsGetCurrentContext() 来得到最顶层的上下文。你可以使用 UIGraphicsPushContext() 和 UIGraphicsPopContext() 在 UIKit 的堆栈中推进或取出上下文。

最为突出的是，UIKit 使用 UIGraphicsBeginImageContextWithOptions() 和 UIGraphicsEndImageContext() 方便的创建类似于 CGBitmapContextCreate() 的位图上下文。混合调用 UIKit 和 Core Graphics 非常简单：

```
UIGraphicsBeginImageContextWithOptions(CGSizeMake(45, 45), YES, 2);
CGContextRef ctx = UIGraphicsGetCurrentContext();
CGContextBeginPath(ctx);
CGContextMoveToPoint(ctx, 16.72, 7.22);
CGContextAddLineToPoint(ctx, 3.29, 20.83);
...
CGContextStrokePath(ctx);
UIGraphicsEndImageContext();
--------------->或者另外一种方法:
CGContextRef ctx = CGBitmapContextCreate(NULL, 90, 90, 8, 90 * 4, space, bitmapInfo);
CGContextScaleCTM(ctx, 0.5, 0.5);
UIGraphicsPushContext(ctx);
UIBezierPath *path = [UIBezierPath bezierPath];
[path moveToPoint:CGPointMake(16.72, 7.22)];
[path addLineToPoint:CGPointMake(3.29, 20.83)];
...
[path stroke];
UIGraphicsPopContext(ctx);
CGContextRelease(ctx);
```

## 动画
iOS制作动画有两种方式。
### 一种是用CoreAnimation提供的CALayer属性，他们支持动画效果。
对于属性支持的动画有隐式动画和显式动画。

1. 隐式动画 : 当修改 CALayer 的支持动画的属性时，会产生隐式动画，效果是平滑过渡的。但修改UIView的不会。这种叫隐式动画。是系统在RunLoop每次循环时都会收集一次 CALayer 图层树的改变，用CATransaction提交到视图控制器。产生动画。可以如下方式修改默认的时间等动画配置。UIView 的 `beginAnimations:context `和`+commitAnimations`都是对CATransaction 的封装。

```
	 [CATransaction begin];
    [CATransaction setAnimationDuration:1.0];
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    [CATransaction commit];
```

2. 显式动画 : 主动创建CABasicAnimation对象，描述属性值得变化以及动画参数，并添加到CALayer

```
- (IBAction)changeColor
{
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor;
    animation.delegate = self;
    [self.colorLayer addAnimation:animation forKey:nil];
}
```

3. 通过自定义属性 实现可见动画 ：
	
```
-------------> ClockFace时钟声明一个属性在动态的去添加.
@interface ClockFace: CAShapeLayer
@property (nonatomic, strong) NSDate *time;
@end

@implementation ClockFace
@dynamic time;  //动态方式创建time
- (id)init
{
    if ((self = [super init]))
    {
        self.bounds = CGRectMake(0, 0, 200, 200);
    }
    return self;
}

+ (BOOL)needsDisplayForKey:(NSString *)key  // 告诉CALayer，time属性变化时需要更新Display
{
    if ([@"time" isEqualToString:key])
    {
        return YES;
    }
    return [super needsDisplayForKey:key];
}

- (id<CAAction>)actionForKey:(NSString *)key  对这个属性做差值。
{
    if ([key isEqualToString:@"time"])
    {
        CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:key];
        animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
        animation.fromValue = @([[self presentationLayer] time]);
        return animation;
    }
    return [super actionForKey:key];
}

- (void)display
{
    NSLog(@"time: %f", [[self presentationLayer] time]);//得到动态的差值
    //2014-04-28 22:43:31.200 ClockFace[49176:60b] time: 0.000000
    //2014-04-28 22:43:31.203 ClockFace[49176:60b] time: 0.002894
    //2014-04-28 22:43:31.263 ClockFace[49176:60b] time: 0.363371
    //2014-04-28 22:43:31.300 ClockFace[49176:60b] time: 0.586421
    //....
    //2014-04-28 22:43:31.427 ClockFace[49176:60b] time: 1.352016
    //2014-04-28 22:43:31.446 ClockFace[49176:60b] time: 1.460729
    //2014-04-28 22:43:31.464 ClockFace[49176:60b] time: 1.500000
}
@end

```
	
4. 通过自定义属性 实现非可见动画 ：其实上面的time，也可以是一个Audio的volume。这样设置了中间差值后声音会一点点的到达目标。

### 第二种复杂的不能用属性实现的动画就需要CoreGraphic来不断绘制，实现动画效果。

1. 创建 CADisplayLink 定时器 以1/60秒的频率 对自定义动画的CALayer或UIView调用 setNeedsDisplay 方法标记需要被刷新，下一次runloop会被drawrect。
	+ 重新绘制自己 layer.content 
	+ 创建 CAShapeLayer 添加到自己的layer上。并用UIBezierPath创建不同的绘制路径给CAShapeLayer，使CAShapeLayer产生动画。



+ [Layer 中自定义属性的动画](https://objccn.io/issue-12-2/)

## 转场动画
+ 非交互动画
	+ 实现`UIViewController`的代理`<UIViewControllerTransitioningDelegate>`方法对象，`TransitionDelegate`，并在代理方法中把`Animator`通过`animationControllerForPresentedController`和`animationControllerForDismissedController`方法传递给系统。
	+ `Animator` 实现了 `UIViewControllerAnimatedTransitioning` 转场动画的协议。
		+ func transitionDuration(transitionContext: UIViewControllerContextTransitioning?) -> NSTimeInterval  //持续时间
		+ func animateTransition(transitionContext: UIViewControllerContextTransitioning)	// 在这里拿到fromView和toView实现动画

+ 交互式动画
	+ 假如是交互式动画，`UIViewController`还会对`TransitionDelegate`代理发送`interactionControllerForPresentation`和`interactionControllerForDismissal`，来获得用于获取动画交互信息的对象`InteractionController`
	+ `InteractionController` 实现 `UIPercentDrivenInteractiveTransition` ：`UIViewControllerInteractiveTransitioning` 转场交互式动画协议
		+ (void)startInteractiveTransition:(id <UIViewControllerContextTransitioning>)transitionContext; // 可以拿到动画的上下文信息。
		+ 通过手势得出来的偏移量，调用`updateInteractiveTransition` 来更新百分比。

+ Push 和 Present的方式都支持！
+ [iOS自定义转场动画实战讲解](https://bestswifter.com/custom-transition-animation/)
+ [自定义 ViewController 容器转场](https://bestswifter.com/custom-transition-animation/)
## AsyncDisplay优势

阻塞主线程的绘制任务主要是这三大类：Layout计算视图布局文本宽高、Rendering文本渲染图片解码图片绘制、UIKit对象创建更新释放。除了UIKit和CoreAnimation相关操作必须在主线程中进行，其他的都可以挪到后台线程异步执行。

AsyncDisplay通过抽象UIView的关系创建了ASDisplayNode类，ASDisplayNode是线程安全的，它可以在后台线程创建和修改。Node 刚创建时，并不会在内部新建 UIView 和 CALayer，直到第一次在主线程访问 view 或 layer 属性时，它才会在内部生成对应的对象。当它的属性（比如frame/transform）改变后，它并不会立刻同步到其持有的 view 或 layer 去，而是把被改变的属性保存到内部的一个中间变量，稍后在需要时，再通过某个机制一次性设置到内部的 view 或 layer。从而可以实现异步并发操作。

AsyncDisplay实现依赖如同Core Animation在runloop中注册observer事件来触发。 


## YYKit 优化TableView

1. 预排版，后台拿到Json后生产CellLayout存起来，不要用Autolayout。
2. 预渲染，自己绘制头像圆角和阴影等，避免离屏渲染
3. 使用YYAsyncCALayer 还可以异步绘制。

+ [iOS核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/chapter8/property-animations.html)
+ [AsyncDisplay](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
+ [绘制像素到屏幕上](https://objccn.io/issue-3-1/)
+ [iOS 事件处理机制与图像渲染过程](https://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=400417748&idx=1&sn=0c5f6747dd192c5a0eea32bb4650c160&scene=4#wechat_redirect)
