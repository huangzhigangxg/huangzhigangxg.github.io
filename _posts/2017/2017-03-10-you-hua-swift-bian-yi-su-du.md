---
layout: post

title: 优化 Swift 编译速度

date: 2016-11-17 15:32:24.000000000 +09:00

---
# 优化 Swift 编译速度

## 查看编译时间

编译速度慢的原因是swift的类型推断机制，当类型复杂时，数据结构多时，
编译速度就会变得很慢，可以帮助swift编译器识别变量类型 ：
方法如下：

通过设置 Build Settings 中的 set Other Swift Flags 添加 -Xfrontend -debug-time-function-bodies 可以显示编译模块的时间 ，并且把结果排序输出到文本中：

> xcodebuild -workspace App.xcworkspace -scheme App clean build  | grep .[0-9]ms | grep -v ^0.[0-9]ms | sort -nr > culprits.txt


之后再帮助编译器识别变量类型：
let myCompany: Dictionary<String, AnyObject> = 。。。。



+ [参考：1](https://thatthinginswift.com/debug-long-compile-times-swift/)
+ [参考：2](http://irace.me/swift-profiling)

## Optimization Level

这个是xcode Built Setting里的一个参数，Optimization Level是指编译器的优化层度

+ None: 编译器不会尝试优化代码，当你专注解决逻辑错误、编译速度快时使用此项。
+ Fast: 编译器执行简单的优化来提高代码的性能，同时最大限度的减少编译时间，该选项在编译过程中会使用更多的内存。
+ Faster: 编译器执行所有优化，增加编译时间，提高代码的性能。
+ Fastest: 编译器执行所有优化，改善代码的速度，但会增加代码长度，编译速度慢。
+ Fastest, Smallest: 编译器执行所有优化，不会增加代码的长度，它是执行文件占用更少内存的首选方案

所以说我们平时开发的时候可以选择使用None来不给代码执行优化，这样既可以减少编译时间，又可以看出你代码哪里有性能问题。

而你的release版应该选择Fastest, Smalllest，这样既能执行所有的优化而不增加代码长度，又能使执行文件占用更少的内存。

### pod里的Optimization Level

```
post_install do |installer|
  installer.pods_project.build_configurations.each do |config|
    if config.name.include?("Dev")
      config.build_settings['GCC_OPTIMIZATION_LEVEL'] = '0'
    end
  end
end
```

## 设置xcode编译的线程数

```
defaults write xcodebuild PBXNumberOfParallelBuildSubtasks 8
defaults write xcodebuild IDEBuildOperationMaxNumberOfConcurrentCompileTasks 8
defaults write com.apple.xcode PBXNumberOfParallelBuildSubtasks 8
defaults write com.apple.xcode IDEBuildOperationMaxNumberOfConcurrentCompileTasks 8
```

XCode默认使用与CPU核数相同的线程来进行编译，但由于编译过程中的IO操作往往比CPU运算要多，因此适当的提升线程数可以在一定程度上加快编译速度。


## Debug Information Format

在工程对应Target的Build Settings中，找到Debug Information Format这一项，将Debug时的DWARF with dSYM file改为DWARF。 其实Debug Information Format就是表示是否生成.dSYM文件，也就是符号表

## 将Build Active Architecture Only改为Yes

这一项设置的是是否仅编译当前架构的版本，如果为No，会编译所有架构的版本。需要注意的是，此选项在Release模式下必须为No，否则发布的ipa在部分设备上将不能运行。这一项更改完之后，可以显著提高编译速度。


```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
        if config.name == 'Dev'
          config.build_settings['ONLY_ACTIVE_ARCH'] = 'YES'
        end
    end
  end
end
```