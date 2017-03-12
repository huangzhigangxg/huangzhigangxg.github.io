---
layout: post

title: RunTime实践：无痕迹埋点

date: 2017-01-11 15:32:24.000000000 +09:00

---

##在用swizzle方法 监听原生类的事件接口时，比如说：

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