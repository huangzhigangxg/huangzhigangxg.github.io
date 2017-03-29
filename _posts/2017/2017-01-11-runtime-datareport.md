---
layout: post

title: RunTime实践：分离埋点代码

date: 2017-01-11 15:32:24.000000000 +09:00

---

## 监测界面的出现和消失
对UIViewController用swizzle方法监测 viewDidAppear 和 viewDidDisappear 两个消息；
并为UIViewController扩展两个属性，dataReport_pageUUID 和 dataReport_appearTime。

```Objective-C
@implementation UIViewController(DataReport)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [UIViewController swizzleSelector:@selector(viewDidAppear:) new:@selector(dataReport_UIViewController_ viewDidAppear:)];
        [UIViewController swizzleSelector:@selector(viewDidDisappear:) new:@selector(dataReport_UIViewController_ viewDidDisappear:)];
    });
}

- (void)setDataReport_pageUUID:(NSString *)dataReport_pageUUID
{
    objc_setAssociatedObject(self, @selector(dataReport_pageUUID), dataReport_pageUUID, OBJC_ASSOCIATION_COPY);
}

- (NSString *)dataReport_pageUUID
{
    return objc_getAssociatedObject(self, @selector(dataReport_pageUUID));
}
- (void)setDataReport_appearTime:(NSDate *)dataReport_appearTime
{
    objc_setAssociatedObject(self, @selector(dataReport_appearTime), dataReport_appearTime, OBJC_ASSOCIATION_COPY);
}

- (NSDate *)dataReport_appearTime
{
    return objc_getAssociatedObject(self, @selector(dataReport_appearTime));
}

- (void)dataReport_UIViewController_viewDidAppear:(BOOL)animated
{
    [self dataReport_UIViewController_viewDidDisappear:animated];
    
    self.dataReport_pageUUID = [UIViewController checkPageUUID:self];
	
	//....
}
@end

```
## 监测按钮事件的点击响应
对UIControl用swizzle方法监测 init , initWithFrame ,initWithCoder 消息；在生成UIControl对象时，给自己添加个监测响应事件。
为UIControl扩展两个属性 dataReport_widgetID 和 dataReport_info；

```Objective-C
@implementation UIControl(DataReport)

- (void)setDataReport_info:(NSDictionary *)dataReport_info
{
    objc_setAssociatedObject(self, @selector(dataReport_info), dataReport_info, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSDictionary *)dataReport_info
{
    return objc_getAssociatedObject(self, @selector(dataReport_info));
}

- (void)setDataReport_widgetID:(NSString *)dataReport_widgetID
{
    objc_setAssociatedObject(self, @selector(dataReport_widgetID), dataReport_widgetID, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)dataReport_widgetID
{
    return objc_getAssociatedObject(self, @selector(dataReport_widgetID));
}

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [UIControl swizzleSelector:@selector(init) new:@selector(dataReport_init)];
        [UIControl swizzleSelector:@selector(initWithFrame:) new:@selector(dataReport_initWithFrame:)];
        [UIControl swizzleSelector:@selector(initWithCoder:) new:@selector(dataReport_initWithCoder:)];
    });
}
- (instancetype)dataReport_init
{
    id o = [self dataReport_init];
    [o addTarget:o action:@selector(dataReport_action:) forControlEvents:UIControlEventTouchUpInside];
    return o;
}
- (instancetype)dataReport_initWithFrame:(CGRect)frame
{
    id o = [self dataReport_initWithFrame:frame];
    [o addTarget:o action:@selector(dataReport_action:) forControlEvents:UIControlEventTouchUpInside];
    return o;
}
- (nullable instancetype)dataReport_initWithCoder:(NSCoder *)coder
{
    id o = [self dataReport_initWithCoder:coder];
    [o addTarget:o action:@selector(dataReport_action:) forControlEvents:UIControlEventTouchUpInside];
    return o;
}
- (void)dataReport_action:(id)sender
{
    if (self.dataReport_widgetID)
    {
        [DataReport reportEventData:self.dataReport_widgetID paras:self.dataReport_info];
    }
}
@end
```


## 在用swizzle方法 监听原生类的事件接口时 碰见的问题：

>[UIViewController swizzleSelector:@selector(viewWillAppear:) new:@selector(dataReport_viewWillAppear:)];

这里可以监听所有UIViewController的出现用于记录log。但是某些UIViewController的子类比如说

>[SearchViewController swizzleSelector:@selector(viewWillAppear:) new:@selector(dataReport_viewWillAppear:)];

用于记录SearchViewController界面更具体的log信息。
就不能用同样的方法名（dataReport_viewWillAppear） 因为会造成调用循环，过程是这样：

+ 1. 当 SearchViewController.dataReport_viewWillAppear 被调用，
+ 2. 紧接着是SearchViewController.viewWillAppear被调用 ， 
+ 3. viewWillAppear 里面有 super.viewWillAppear 
所以UIViewController.dataReport_viewWillAppear会被调用.
+ 4. 然后 递归开始！因为当前self调用的是SearchViewController，isa指针指向的是SearchViewController，会首先从SearchViewController中查找dataReport_viewWillAppear的SEL（子类的方法优先被找到）,然后调用。就会回到紧接着是SearchViewController.viewWillAppear。
所以造成了调用循环

## 解决方法
+ 1. *方法一*加上方法前缀，最后一步就会找到父类UIViewController的方法。
  [SearchViewController swizzleSelector:@selector(viewWillAppear:) new:@selector(dataReport_searchViewController_viewWillAppear:)];
+ 2. *方法二*用C函数声明的方式 实现IMP指针，直接替换IMP ，而不用给原生的类UIViewController添加新的SEL（dataReport_viewWillAppear），就不会发生循环调用。如下举例：

```
static void (*InitWithFrameIMP)(id self, SEL _cmd, NSCoder *decoder);

static id MyInitWithFrame(id self, SEL _cmd, NSCoder *decoder) {
    // do custom work
    NSLog(@"In UIImageView(interceptInit) MyInitWithFrame");
    InitWithFrameIMP(self, _cmd, decoder);
    return self;
}

+ (void)load {
    [self swizzle:@selector(initWithFrame:) with:(IMP)MyInitWithFrame store:(IMP *)&InitWithFrameIMP];
｝
  
@implementation NSObject (FRRuntimeAdditions)
+ (BOOL)swizzle:(SEL)original with:(IMP)replacement store:(IMPPointer)store {
    return class_swizzleMethodAndStore(self, original, replacement, store);
}
@end

  BOOL class_swizzleMethodAndStore(Class class, SEL original, IMP replacement, IMPPointer store) {
    IMP imp = NULL;
    Method method = class_getInstanceMethod(class, original);
    if (method) {
        const char *type = method_getTypeEncoding(method);
        imp = class_replaceMethod(class, original, replacement, type);
        if (!imp) {
            imp = method_getImplementation(method);
        }
    }
    if (imp && store) { *store = imp; }
    return (imp != NULL);
}

```