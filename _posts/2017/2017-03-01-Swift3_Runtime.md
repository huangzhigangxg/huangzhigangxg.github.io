---
layout: post

title: Swift语言动态性 运行时相关

date: 2017-03-2 15:34:24.000000000 +09:00

---

## @objc 和 dynamic 
这两个关键字都可以用来修饰 Swift 的 class 和 func ，使它们像OC一样有动态化的能力，又享受了函数表派发的高性能。

+ 纯Swift类没有动态性，但在方法、属性前添加dynamic修饰可以获得动态性。
+ 值得注意的是 ：加了@objc标识的方法、属性无法保证都会被运行时调用，因为Swift会做静态优化。要想完全被动态调用，必须使用dynamic修饰。使用dynamic修饰将会隐式的加上@objc标识。

+ 继承自NSObject的Swift类，其继承自父类的方法具有动态性，其他自定义方法、属性需要加dynamic修饰才可以获得动态性。

```
class MyViewController: UIViewController{
	overide func viewDidLoad(){   //继承OC的方法 默认前面有 @objc
	}
	func setupSubviews(){ //自己后来添加的方法 没有 @objc 
	}
	func loadUrl(){ //自己后来添加的方法 就没有 @objc 
	}
}
```
+ Swift类在Objective-C中会有模块前缀 `id cls = objc_getClass("YourTarget.MyViewController");` 

+ 若方法的参数、属性类型为Swift特有、无法映射到Objective-C的类型(如Character、Tuple)，则此方法、属性无法添加dynamic修饰（会编译错误）

+ [Swift Runtime动态性分析](http://www.infoq.com/cn/articles/dynamic-analysis-of-runtime-swift)

## Swift函数派发机制

编译型语言三种基础的函数派发方式：

+ `直接派发(Direct Dispatch)` 性能最高的方式，直接绑定函数地址，又叫静态编译 `执行耗时1纳秒`
+ `函数表派发(Table Dispatch)` 最常见的方式，类模型里维护一个虚函数表，子类增加新的方法就添加在子类虚函数表的后面，覆盖父类的方法就直接替换函数地址。运行时通过虚函数表决定实际调用的函数。`执行耗时5纳秒`
+ `消息机制派发(Message Dispatch)` 最动态的方式,功能强大，swizzling，甚至改变继承关系，自定义派发，`Objective-C` 会用树来构建这种继承关系，当一个消息被派发, 运行时会顺着类的继承关系向上查找应该被调用的函数. 但效率很低。

>Java 默认使用函数表派发, 但你可以通过 final 修饰符修改成直接派发. C++ 默认使用直接派发, 但可以通过加上 virtual 修饰符来改成函数表派发. 而 Objective-C 则总是使用消息机制派发, 但允许开发者使用 C 直接派发来获取性能的提高. 

而Swift如何选择一个函数的派发机制跟下面四个因素有关：

+ 声明的位置 
   + 值类型总是会使用直接派发, 简单易懂
   + 而协议和类的 extension 都会使用直接派发
   + NSObject 的 extension 会使用消息机制进行派发
   + NSObject 声明作用域里的函数都会使用函数表进行派发.
   + 协议里声明的, 并且带有默认实现的函数会使用函数表进行派发
+ 引用类型  
   + 结构体会使用直接派发，但是当他符合某个协议，用协议的类型去调用协议里声明的方法，会使用函数表进行派发，如果协议里没有声明，而直接在协议扩展里实现了该方法，根据上面声明功能位置的说法，选择直接派发。
+ 特定的行为 
	+ final 使任何类里面的函数失去动态性，变成直接派发
	+ dynamic 可以让类里面的函数使用消息派发
	+ @objc & @nonobjc 显式地声明了一个函数是否能被 Objective-C 的运行时捕获到.
	+ final @objc 调用函数的时候会使用直接派发, 但也会在 Objective-C 的运行时里注册响应的 selector. 
	+ @inline 告诉编译器使用直接派发

+ 显式地优化(Visibility Optimizations)
	+ 一个函数从来没有被继承 override, Swift 就会检测使用直接派发. eg : `private func doSomething()`
	+ 不加 dynamic 修饰的变量，KVO的回调函数不会被触发 eg : `var age = Int(1)`

![](/assets/images/14826580751015.png)

+ [深入理解 Swift 派发机制](https://kemchenj.github.io/2017/01/09/2016-12-25-1/)


+ ios方法来源
	+ Objetive-C  `Objetive-C 统一的Message`
		+ Class 
		+ Category : @interface UIImage (GIF) 将类的功能分开写，运行时加载
		+ Extension : @interface Person (）不指定名字,用于指定私有方法 编译期确定
	+ Swift
		+ Class 
			+ SwiftClass `Table`
				+ Extension  `Direct`
				+ ProtocolExtension `Direct`
			+ NSObjectSubClass  `Table`
				+ NSObjectSubClass Extension `Message`
				+ NSObjectSubClass ProtocolExtension `Direct`
		+ Struct／enum `swift值类型都是Direct`
			+ SwiftStruct `Direct`
			+ Extension `Direct`
			+ ProtocolExtension`Direct`
来看一例子：

```Swift
struct Pizza {
  let ingredients: [String]
}
  
protocol Pizzeria {
  func makePizza(ingredients: [String]) -> Pizza
  func makeMargherita() -> Pizza
}
  
extension Pizzeria {
  func makeMargherita() -> Pizza {
    return makePizza(["tomato", "mozzarella"])
  }
}

struct Lombardis: Pizzeria {
  func makePizza(ingredients: [String]) -> Pizza {
    return Pizza(ingredients: ingredients)
  }
  func makeMargherita() -> Pizza {
    return makePizza(["tomato", "basil", "mozzarella"])
  }
}
//1
let lombardis1: Pizzeria = Lombardis()
let lombardis2: Lombardis = Lombardis()
  
lombardis1.makeMargherita()//["tomato", "basil", "mozzarella"]
lombardis2.makeMargherita()//["tomato", "basil", "mozzarella"]
这两个方法都会打印["tomato", "basil", "mozzarella"]，原因是根据上面说的方法的声明位置决定了他的派发方式，makeMargherita()这个方法在protocol Pizzeria有声明，在extension Pizzeria有实现，所以编译器在处理这个函数的派发方式时采用函数表Table的方式，在运行时决定的具体实现哪个函数，而运行时，类型强转已经失去效果（类型强转只给编译器看的），他们还是原来的struct Lombardis结构体，走他自己的Table实现。也就是//["tomato", "basil", "mozzarella"]。

假如Pizzeria协议没有声明makeMargherita()方法，但是扩展中仍然提供了如下的代码的这个方法默认的实现，会发生什么？

protocol Pizzeria {
  func makePizza(ingredients: [String]) -> Pizza
}
  
extension Pizzeria {
  func makeMargherita() -> Pizza {
    return makePizza(["tomato", "mozzarella"])
  }
}

lombardis1.makeMargherita()//["tomato", "mozzarella"]
lombardis2.makeMargherita()//["tomato", "basil", "mozzarella"]

根据上面的图，当方法声明在Protocol的里时，采用Table方式，而这里去掉了protocol Pizzeria里的声明，仅有extension Pizzeria，同样根据上图，看出Pizzeria这个方法makeMargherita是直接静态编译，所以编译器在处理这个函数的派发方式时采用静态编译的方式 也就是说，把Pizzeria的makeMargherita()方法直接绑定到被强转的lombardis1上。所以才会打印["tomato", "mozzarella"]
```
		
 `Direct` 的直接编译会根据当前类型指针，在编译器直接绑定方法
 
 `Table` 则会产生 Extension 覆盖原生class或者struct接口的覆盖

## Swift 性能优化
### 性能的因素因素
+ 内存分配 ：
	 + 栈,移动指针就可以实现对象的`allocate`和`deallocate`,速度快
	 + 堆,先搜索足够大小空间，还要保证线程之间共享安全，速度慢
+ 引用计数 ：
	 + 使用维护对象的引用计数，retain和release操作频繁
+ 调度与对象 ：
	 + 静态调度，函数直接绑定实现，编译器可以更大程度优化成内联函数 
    + 动态调度，运行时查找函数实现，编译器无法优化

### 尽可能使用值类型，代替引用类型 

![](/assets/images/WX20170321-184339@2x.png)

+ 值语意修改或者拷贝不容易出错，线程安全，不需要引用计数
+ class 引用类型 需要编译器频繁插入 retain(),relesae()
+ 小的结构体能在栈上內联，栈操作速度快，class引用类型，用堆操作速度慢。
还需要注意的是，String虽然是Struct类型，但是真正的存储区storage还是在堆上，不完全是纯值类型。所以尽量选用更小的类型。

```
struct Attachment {	let fileURL: URL	let uuid: String	let mimeType: String
}

--------> 改成：

struct Attachment {	let fileURL: URL	let uuid: UUID //比String更简单的类型	let mimeType: MimeType //枚举
}
```


### 使用(范型+协议)实现静态多态，代替(协议/继承)实现的动态多态

![](/assets/images/WX20170321-185410@2x.png)

上面是靠继承实现的动态多态，每一个对象的内存中有一个指针指向静态内存区的typeInfo表。在运行时通过每一个对象的typeInfo虚函数表找到方法的实现

![](/assets/images/WX20170321-190302@2x.png)

![](/assets/images/WX20170321-191131@2x.png)

上面是靠协议实现的动态多态，每一个实现类Drawable协议的对象，都会被一个叫做`即有容器 Existential Container`的盒子封装起来，这个容器占有5个word，前3个word是value buffer，可以放下小于3个word的结构体，如果大于3个word，就会放在堆中，用第一word存放堆里的地址，第4个word存放一个 `方法列表指针 Value Witness Table(VWT)` 里面是用于在堆上管理结构体的生命周期的方法 `allocate:copy:destruct:deallocate:`. 第五个word也存放一个方法列表 `Protocol Witness Table`,里面是对象实现目标协议的具体实现函数。运行时通过查找 `即有容器 Existential Container` 的 `Protocol Witness Table` 完成函数的调用。

![](/assets/images/WX20170321-191547@2x.png)

上面的代码表述了编译器是如何通过协议参数构建`即有容器 Existential Container`并调用正确方法的。

1. 生产一个既有容器`ExistContDrawable()` local
2. 将参数val的vwt和pwt都赋值给既有容器local
3. 调用vwt.allocateBufferAndCopyValue，将val的值拷贝到local上。
4. 调用vwt.projectBuffer,拿到local的对象值。也就是val传给pwt.draw，完成调用。
5. 最后调用vwt方法列表里的方法，清空local里的值。

上面都是动态的多态实现方法。下面看看 效率更高的 范型+协议实现的静态多态。

![](/assets/images/WX20170321-214652@2x.png)

![](/assets/images/WX20170321-215353@2x.png)

编译器通过PWT和VWT足够的信息，通过一份代码，转换成多个类型的版本，供调用者静态调用。
由于是静态编译，上面的代码还会被优化。

```
func drawACopyOfAPoint(local : Point) {	local.draw()}func drawACopyOfALine(local : Line) {	local.draw()}drawACopyOfAPoint(Point(…))drawACopyOfALine(Line(…))

------> 优化成

Point().draw()Line().draw()

```

### 使用 Copy On Write 处理容量大的值类型。

当自定义的Struct的容量很大时拷贝成本很高，采用Copy On Write方式提高struct的读写性能。在内置类型String,Array,Set,Dictionary均使用了这个技术。

![](/assets/images/WX20170321-220708@2x.png)

1. 用一个LineStorage引用类型在堆上做真正的存贮容器。
2. 当 Struct Line值类型 发生拷贝时`值语意拷贝 let lina2 = line1`，仅仅是LineStorage引用类型容器引用计数+1。
3. 只有调用 mutating方法修改自身时，判断 引用计数是否大于1.然后重新生成一份LineStorage赋给容器变量storage，之后再去修改成想要的结果。

+ [真实世界中的 Swift 性能优化](https://realm.io/cn/news/real-world-swift-performance/?w=1)
+ [wwdc2016](https://developer.apple.com/videos/play/wwdc2016/416/)