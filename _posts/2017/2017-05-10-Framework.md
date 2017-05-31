---
layout: post

title: Framework

date: 2017-05-10 15:34:24.000000000 +09:00

---



## 理清概念
	
+ 静态库：编译期就绑定到拷贝到可执行文件中，容易拷贝多份，不能共用
+ 动态库：运行时系统动态加载到内存，可多个进程共用 
+ 二进制库他们的好处都有模块化，分工合作，减少编译时间，可重用。

## 适用情况
+ 动态库只允许ios8+，而且目前Swift只支持动态库，所以要想用Swift版的动态库，就必须要求设备在8.0以上
+ Swift本身语言是从Swift1.0开始就支持最低ios7.0的设备了。但是Swift目前只支持动态库。
+ 也就是说 要想支持ios7的设备，上使用二进制管理代码就必须用OC写静态库。

## 注意事项

### Framework的资源管理
#### 是否需要Bundle？
以前的静态库不能包含资源，使用Bundle将不同种类的资源管理在一个文件中是必须的，但Framework自身就可以很好地管理资源，就不用单独建一个Bundle来额外的存贮资源，除非一些 大型的地图，视频之类，不随着Framework一起发布的，可以使用Bundle单独封装。
####  Framework开发获取的资源的方式有所不同。
+ 用类确定哪个框架，从而锁定哪个框架的`Bundle`。
+ 因为是多个模块一起开发，每个`Framework`通常想要获取自己的`Bundle` 资源时，需要通过 `Bundle(for: SomeClassIn Framework.self)` 将`Framework`框架中的一个`class`传进去，，才能获取这个`class`所在的Bundle。

```Swift
//----->在子工程中使用自己的图片
        let bundle = Bundle(for: LoginView.self)
        let image = UIImage(named: "WechatIMG1", in: bundle, compatibleWith: nil )        
        self.view.backgroundColor = UIColor(patternImage:image!)
        
//----->在主工程中获取子工程的storyboard
        let storyBoard = UIStoryboard(name: "Storyboard", bundle: loginBundle)
        let vc = storyBoard.instantiateInitialViewController()!
        self.navigationController?.pushViewController(vc, animated: true)
 
```
+ [framework中关于资源的读取](http://www.jianshu.com/p/3549984315bf)

#### 静态Framework里不需要加入资源文件

一般如果是静态Framework的话，资源打包进Framework是读取不了的。静态Framework和.a文件都是编译进可执行文件里面的。只有动态Framework能在.app的Framework文件夹下看到，并读取.framework里的资源文件。

你可以用NSBundle * bundel = [[NSBundle mainBundle] bundlePath];得到.app目录，如果是动态库你能在Framework目录下看到这个动态库以及动态库里面资源文件。然后你只要用NSBundle *bundle = [NSBundle bundleForClass:<#ClassFromFramework#>];得到这个动态库的路径就能读取到里面的资源了。 
但是如果是静态库的话，因为编译进了可执行文件里面，你也就没办法读到这个静态库了，你能看到.app下的Framework目录为空。

+ [参考](https://www.zybuluo.com/qidiandasheng/note/603907)

### 封装
1. Framework开发要做好接口的权限管理.
	+ Publie 修饰的 class，func 或者 var ，能被模块外部访问，最高的级别
	+ Internal 修饰的 class，func 或者 var ，仅能被模块内部访问 ，默认的级别
	+ Private 修饰的 func 或者 var ，仅能被自己的类访问，最低的级别，extension也不行。
	+ open 修饰的类，还能继承。修饰的方法，还能覆盖override
	+ fileprivate 和 private 区别就是一个文件内可访问 包括extension ，一个类内部可访问，不包括extension。Swift中经常用extension来分离代码，用fileprivate可以更好导出给当前文件的extension用。
	+ open > public > interal > fileprivate > private   

2. 注意属性的限制 readonly
3. final ：用在 class，func 或者 var 前面进行修饰，表示不允许对该内容进行继承或者重写操作

#### Umbrella Header在framework
+ 可以添加子framework。但是还没查清楚怎么用
+ 其次就是可以更规范的引入framework master header.框架主头文件， 保护framework里其他头文件不被引用。
+ [iOS - Umbrella Header在framework中的应用](http://blog.startry.com/2015/08/25/Renaming-umbrella-header-for-iOS-framework/)

### API命名
+ 使用好时态？
	+ 有ing和ed后缀的时候 表明这是一个名词，是有返回值的。如果不包含这些后缀的时候 则表明这是一个动词，要修改自身的值，修改某块内存 
		+ customArray.reverse()   //对自己的元素翻转，没有返回值 	 	+ customArray.reversed() //Copy一个数组并翻转其内部元素，然后作为返回值
		+ customArray.appending(item) //表示copy一个数组后，在新的里添加item后作为返回值返回 	
+ 使用好介词: at,in,of
	+ `remove(at: Index) -> Element` > `remove(i: Int) -> Element` remove index or element? 
+ 使用好副词 
	+ `recursivelyFetch(urls: [(String, Range<Version>)]) throws -> [T]` > `fetch(urls: [(String, Range<Version>)]) throws -> [T] // <- how?` 很好的修饰了动作是递归获取。
+ 方法名应该是动词或者动词短语开头
+ 属性名应该是名词 

### Swift可以通过 module 来提供命名空间

```
MyClass.hello()
// hello from app

MyFramework.MyClass.hello()
// hello from framework
```
+ [命名空间](http://swifter.tips/namespace/)

### 注意系统api版本的判断

### 删除Framework中无用的mach-O文件来瘦身
+ 拆分Framework里面的mach-O，将无用的删除。
+ [iOS瘦身之删除无用的mach-O文件](http://mp.weixin.qq.com/s?__biz=MzA4MjA0MTc4NQ==&mid=504089563&idx=1&sn=9328409758177cb6df3edbb3b8a0c161#rd)


### OC和Swift混编
+ Target为顶层应用
	+ OC给Swift用：因为Swift不用引入头文件，所以需要OC的头文件要找个地方放，就是在 Spottly-Bridging-Header.h 
	+ Swift给OC用：因为OC需要引入头文件，所以Xcode自动生成个 Spottly-Swift.h。让在OC在文件中引入，就能用。注意要
+ Target为Framework
	+ OC给Swift用：不把OC放进Spottly-Bridging-Header.h里了，而是 SpottlyFramework.h里，他就是这个Framework的umbrella header。
	+ Swift给OC用：将Xcode自动生成的Spottly-Swift.h导入oc中，还是和Target为顶层应用一样。
+ [OC和Swift混编](http://c.biancheng.net/cpp/html/2268.html)
+ [Apple doc](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html#//apple_ref/doc/uid/TP40014216-CH10-XID_84)

### 静态库链接符号冲突
#### 产生的原因
首先确认一点，连接器连接到相同的符号时一定会报错 `duplicate symbol`,但是为什么自己写的A/B两个静态库包含同样的方法，同时连接进Demo中，调用他们都有方法却没报错呢? 	

```
//------------>静态库：LibraryA
void sayHello(){
    NSLog(@"hello from LibraryA");
}

@implementation SameObject
+ (void)hello{
    NSLog(@"hello from LibraryA");
}
@end
//------------>静态库：LibraryB
void sayHello(){
    NSLog(@"hello from LibraryB");
}
@implementation SameObject
+ (void)hello{
    NSLog(@"hello from LibraryB");
}
@end

//------------>在main工程中同时引用静态库：LibraryA 和 LibraryB
#import "LibraryA.h"
#import "LibraryB.h"

- (void)viewDidLoad {
    [super viewDidLoad];
    sayHello(); //`duplicate symbol`
//    [SameObject hello]; //Duplicate interface definition for class
}
```
由于LibraryA和LibraryB都包含sayHello的符号，应该会报出 `duplicate symbol`的错误才对，但是编译通过，执行也正常执行，哪一个先连接，就调用哪一个的实现。`总是调用先连接的方法`。
那么为什么会没有报错呢？因为连接器默认情况下是只连接被调用的符号，也就是说只有遇见了sayHello的符号才会主动去找sayHello符号的实现，而不所有的文件里面所有的符号都链接进去，那么当他编译viewController这个文件时，发现需要连接sayHello，就开始从依次连接顺序的静态库中找这个符号，假如LibraryA先被连接，就直接LibraryA的函数地址交给sayHello，找到了sayHello就不会再去其他静态库再找sayHello。LibraryB的sayHello也就没有被连接器发现，没有被连接到app中。所以没有发生错误。但是有可能LibraryB的其他符号被链接器连接时发现了sayHello的存在，那就很有可能会报错。 或者我们设置连接符号 -all_load后，连接全部符号，就发现LibraryA和LibraryB相同符号的冲突。直接报错。

![设置-all_load后，连接全部符号，就发现了相同符号的冲突](/assets/images/WX20170514-183050@2x.png)
[参考](http://www.jianshu.com/p/782e1786c67a)

#### 解决办法
+ 命名前缀
	+ 在OC中给原本的类NSObject，添加category，也要加前缀。
	+ 自己的Framework依赖其他第三方库，最好使用源码，并在源码的类名前加上自己的前缀，[用脚本给流行的第三方源码添加前缀](http://www.jianshu.com/p/9793dc5a9632)
+ 去掉冲突的部分Mach(.o)
	+ MachOView查看冲突的部分
	+ 解压所有libA.a中的.o文件，`ar xv libA.a`
	+ 删除掉冲突的部分后再重新打包在一起 ` ar rcs libA.a libAFolder/*.o`
	+ [用ar工具 从静态库中删掉冲突的部分](https://itony.me/674.html)
+ 利用编译器选项 准确控制连接目标，避免产生一样的符号
	+ [如果可能的话通过 改用-force_load链接指定的.a静态库，不使用-Objc/-All_load 处理全部静态库，这样可能会引起多个.a的符号冲突](http://www.cnblogs.com/rayshen/p/5160218.html)

### 奇怪问题

1. 正确方式引入了Framework了。#import也能补全。但编译时就是提示找不到头文件。
	+ 可以删掉DerivedData。重新打开一个xcode试一试

+ [iOS专题2:静态库和动态库详解](http://www.jianshu.com/p/c8366e4f9378)
+ [组件化-动态库实战](http://www.cocoachina.com/ios/20170427/19136.html)
+ [静态库与动态库的使用](https://www.gitbook.com/book/leon_lizi/-framework-/details) 
+ [Apple](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/cross_development/Configuring/configuring.html#//apple_ref/doc/uid/10000163i-CH1-SW3)