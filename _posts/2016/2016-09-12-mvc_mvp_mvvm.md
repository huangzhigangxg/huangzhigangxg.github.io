---
layout: post

title: MVC -> MVP -> MVVM 

date: 2016-09-12 15:32:24.000000000 +09:00

---

## MVC

![](/assets/images/93c3aeab-92c0-4cc0-8344-969b668fe76b.png)

MVC 时 View 里面有 观察者Observe 关注Model 里的数据变化。 也就是说View看得见Model

- View是把控制权交移给Controller，自己不执行业务逻辑。
- Controller执行业务逻辑并且操作Model，但不会直接操作View，可以说它是对View无知的。
- View和Model的同步消息是通过观察者模式进行，而同步操作是由View自己请求Model的数据然后对视图进行更新。

### 优点: 

+ View的观察者模式可以实现多个界面同时更新   把业务逻辑全部分离到Controller中

### 缺点:

+ Controller测试困难。因为视图同步操作是由View自己执行，而View只能在有UI的环境下运行。在没有UI环境下对Controller进行单元测试的时候，Controller业务逻辑的正确性是无法验证的View无法组件化。
+ View是强依赖特定的Model的，如果需要把这个View抽出来作为一个另外一个应用程序可复用的组件就困难了。因为不同程序的的Domain Model是不一样的
+ 三个对象 两两存在关系，当改变一个时，都会影响其余的两个，不好维护。

### Apple 理想中的MVC

![](/assets/images/8d779f6a-265b-43c3-90be-dc9997b9963d.png)


+ 与传统的MVC有哪些好处？
	+ 苹果考虑到传统MVC中View依赖Model的问题，将更新View的操作也一并放到Controller里，隔离了View和Model。使他们关系不在是 两两存在关系，view只对Controller负责，model只对Controller负责.


## MVP

![](/assets/images/d8ad72b3-f150-4988-af6f-0db785c40793.png)

MVP 时 打破了View原来对于Model的依赖 全都用Presenter来观察Model变化和处理业务逻辑 , View做的事情很少，`被动的View`。

- View不再负责同步的逻辑，而是由Presenter负责。Presenter中既有业务逻辑也有同步逻辑。
- View需要提供操作界面的接口给Presenter进行调用。（关键）！！！

### 优点：

- 便于测试。Presenter对View是通过接口进行，在对Presenter进行不依赖UI环境的单元测试的时候。可以通过Mock一个View对象，这个对象只需要实现了View的接口即可。然后依赖注入到Presenter中，单元测试的时候就可以完整的测试Presenter业务逻辑的正确性。这里根据上面的例子给出了Presenter的单元测试样例。
- View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务逻辑完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做高度可复用的View组件。

### 缺点：

- Presenter中除了业务逻辑以外，还有大量的View->Model，Model->View的手动同步逻辑，造成Presenter比较笨重，维护起来会比较困难。

### Apple 理想中的MVC和MVP有点像？

+ 确实和MVP有点像，都是将View和Model隔离，将更新view的操作放到了Presenter。但是，Presenter里是不含任何布局代码的。Apple‘s MVC中的Controller里却少不了布局代码。
+ Presenter虽然可以更新View，但他只对一个 `View Api Interface` 接口协议负责，通过这个接口 get/set 数据从View上，由于View是拥有Presenter的，Presenter要写好 View 的 Event 事件处理方法，使View直接对Presenter发送Event。
+ MVP 经常是 M：V：P = 1：1：1的，而 MVC则是 M：V：C = N ：N ：1 ，也就是 MVP中的 m，v，p都是一一对应的关系，颗粒度很细，容易分离重用，一个Presenter 仅仅是包装了一个Model。而MVC中的Controller是一个管理者，同时管理着多个界面，多个model。


## MVVM

![](/assets/images/1b8ff549-4fa4-489a-adf3-e8ba52e6bb96.png)

MVVM 时 VM 把展示逻辑从C中拿出来和业务逻辑从M中拿出来合并到一起 。 并且实现View和ViewModel的双向绑定

### 优点：

- 提高可维护性。以前MVP都是要大量的接口类，来手动的同步View和ViewModel，造成代码量很大。而提供双向绑定机制，减少了代码，提高了代码的可维护性。（关键）！！！
- 简化测试。因为同步逻辑是交由Binder做的，View跟着Model同时变更，所以只需要保证Model的正确性，View就正确。大大减少了对View同步更新的测试。

### 缺点：

- 过于简单的图形界面不适用，或说牛刀杀鸡。
- 对于大型的图形应用程序，视图状态较多，ViewModel的构建和维护的成本都会比较高。
- 数据绑定的声明是指令式地写在View的模版当中的，这些内容是没办法去打断点debug的。


## 总结
MVC中往往 VC 的耦合性比较大。总是成对儿出线。很难单独存在和应用。而V中又有着可能 业务逻辑(push,network,verification,action)和表示逻辑(loaction/city/country)在里面. 而这些在版本迭代更新中是可以继续保留的可以把他们留下啦。仅仅更改界面相关的东西就可以，需要有更轻量级ViewController的策略最好。所以VM层 就出现来。MVVM框架。M层最好还是简单纯数据 。这样方便他被当作值对象在ViewController中传递。

简单的说：MVVM实际上是三层架构，M层（Model实体层）、V层（View表示层，它有DataContext属性，这个属性可以使用DataTemplate模板绑定VM层的数据用来显示）、VM层（ViewModel层，对Model层进行CRUD进行操作，同时对V层提供数据绑定）


- 采用mvvm的好处：项目可测试更高，从而可以执行单元测试；将UI和业务的设计完全分开，View和UnitTest只是ViewModel的两个不同形式的消费者
- Model的职责：主要提供基础实体的属性以及每个属性的验证逻辑；不包含数据的调用；不依赖于任何项目。
- ViewModel：ViewModel是MVVM架构中最重要的部分，ViewModel中包含属性，命令，方法，事件，属性验证等逻辑。为了与View以及Model更好的交互来满足MVVM架构，ViewModel的设计需要注意一些事项或者约束。

ViewModel的属性：ViewModel的属性是View数据的来源。这些属性可由三部分组成：

一部分是Model的复制属性。

另一部分用于控制UI状态。例如一个弹出窗口的控件可能有一个IsClose的属性，当操作完成时可以通过这个属性更改通知View做相应的UI变换或者后面提到的事件通知。

第三部分是一些方法的参数，可以将这些方法的参数设置成相应的属性绑定到View中的某个控件，然后在执行方法的时候获取这些属性，所以一般方法不含参数。

ViewModel的命令：ViewModel中的命令用于接受View的用户输入，并做相应的处理。我们也可以通过方法实现相同的功能。

ViewModel的事件： ViewModel中的事件主要用来通知View做相应的UI变换。它一般在一个处理完成之后触发，随后需要View做出相应的非业务的操作。所以一般ViewModel中的事件的订阅者只是View，除非其他自定义的非View类之间的交互。

ViewModel的方法：有些事件是没有直接提供命令调用的，如自定义的事件。这时候我们可以通过CallMethodAction来调用ViewModel中的方法来完成相应的操作。

View Model一般有以下三个部分组成

+ 1、属性：一个事物，它的类型可以是一个字符型，也可以是一个对象。实现接口INotifyPropertyChanged,那么任何UI元素绑定到这个属性，不管这个属性什么时候改变都能自动和UI层交互。
+ 2、集合：事物的集合，它的类型一般是ObservableCollection，因此，任何UI元素绑定到它，不管这个集合什么时候改变，都可以自动的与UI交互。
　　
　　
+ [MVC—>MVP－>MVVM](https://github.com/livoras/blog/issues/11)
+ [iOS 架构模式 - 简述 MVC, MVP, MVVM 和 VIPER (译)](https://blog.coding.net/blog/ios-architecture-patterns)
+ [你对MVC、MVP、MVVM 三种组合模式分别有什么样的理解](https://www.zhihu.com/question/20148405)
