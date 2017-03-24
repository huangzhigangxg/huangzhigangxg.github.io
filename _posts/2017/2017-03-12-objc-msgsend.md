---
layout: post

title: objc_msgSend 解读

date: 2016-11-17 15:32:24.000000000 +09:00

---

在Objective-C 中，所有的消息传递中的“消息“都会被转换成一个 selector 作为 objc_msgSend 函数的参数：

>[object hello] -> objc_msgSend(object, @selector(hello))

SEL 的定义是 `typedef struct objc_selector *SEL;` objc_selector并没有出现在源码当中，但是

```
	NSLog  (@ "SEL=%s",  @selector (hello ));
	输出 SEL = hello
```
可以推测 SEL 大概可以理解为一个字符串 

将网友调试好的 [OC Runtime 源码包](https://github.com/isaacselement/objc4-706)下载下来，设置断点在 `lookUpImpOrForward `处，然后开始解读objc_msgSend发送顺序

```objective-c
@interface CustomObject : NSObject
-(void)test;
@end

@implementation CustomObject

-(void)test{
    printf("This is CustomObject test func");
}

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        CustomObject *obj = [[CustomObject alloc] init];
        NSLog(@"%p", @selector(test));
        [obj test];
        
    }
    return 0;
}

```
上面是添加的OC类测试代码，下面是`objc_msgSend`调用用来查找对象消息`选择子`的方法。为了提高性能，频繁的获取方法一定会有一个缓存列表，先假设没有缓存的情况，消息是如何转发的。

```c++
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver)
{
    Class curClass;
    IMP imp = nil;
    Method meth;
    bool triedResolver = NO;

    runtimeLock.assertUnlocked();

    // Optimistic cache lookup
    if (cache) {	  //cache参数是NO，直接跳过
        imp = cache_getImp(cls, sel);
        if (imp) return imp;
    }

    if (!cls->isRealized()) { //没实现这个类就将它实现
        rwlock_writer_t lock(runtimeLock);
        realizeClass(cls);
    }

    if (initialize  &&  !cls->isInitialized()) { //没有初始化就初始化这个类
        _class_initialize (_class_getNonMetaClass(cls, inst));
    }

 retry:
    runtimeLock.read(); //上锁读写锁，为了防止缓存区被其他线程刷新
    imp = cache_getImp(cls, sel);
    if (imp) goto done;
    meth = getMethodNoSuper_nolock(cls, sel); //在自身的类里寻找方法,通过传入sel，在自己的methodsList里查找method_t结构体，这里面有IMP指针还有SEL。
    if (meth) { 
        log_and_fill_cache(cls, meth->imp, sel, inst, cls);
        imp = meth->imp;
        goto done;
    }
    curClass = cls;
    while ((curClass = curClass->superclass)) {//找不到，就循环遍历父类。
        imp = cache_getImp(curClass, sel);
        if (imp) {
            if (imp != (IMP)_objc_msgForward_impcache) {
                log_and_fill_cache(cls, imp, sel, inst, curClass);
                goto done;
            }
            else {
                break;
            }
        }

        // Superclass method list.
        //在父类的方法列表里寻找
        meth = getMethodNoSuper_nolock(curClass, sel);
        if (meth) {
            log_and_fill_cache(cls, meth->imp, sel, inst, curClass);
            imp = meth->imp;
            goto done;
        }
    }
	//最终也没有在继承链里找到方法，尝试方法决议。
    if (resolver  &&  !triedResolver) { 
        runtimeLock.unlockRead();
        _class_resolveMethod(cls, sel, inst);//调用 NSObject：+ (BOOL)resolveInstanceMethod:(SEL)sel 方法
        triedResolver = YES;
        goto retry;
    }
    imp = (IMP)_objc_msgForward_impcache; //如果还没有解决，那么将imp设置成_objc_msgForward_impcache函数指针进入消息转发流程。
    cache_fill(cls, sel, imp, inst);//然后加入缓存

 done:
    runtimeLock.unlockRead();

    return imp;
}

```
这个过程总结：

1. 缓存命中
2. 查找当前类的缓存及方法
3. 查找父类的缓存及方法
4. 方法决议
5. 消息转发

+ [objc_msgSend消息传递学习笔记 – 对象方法消息传递流程](http://www.cnblogs.com/fengmin/p/5820453.html)
+ [从源代码看 ObjC 中消息的发送](http://draveness.me/message/)

