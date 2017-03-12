---
layout: post

title: 优化 Swift 编译速度

date: 2016-11-17 15:32:24.000000000 +09:00

---
# 优化 Swift 编译速度

编译速度慢的原因是swift的类型推断机制，当类型复杂时，数据结构多时，
编译速度就会变得很慢，可以帮助swift编译器识别变量类型 ：
方法如下：

通过设置 Build Settings 中的 set Other Swift Flags 添加 -Xfrontend -debug-time-function-bodies 可以显示编译模块的时间 ，并且把结果排序输出到文本中：

xcodebuild -workspace App.xcworkspace -scheme App clean build  | grep .[0-9]ms | grep -v ^0.[0-9]ms | sort -nr > culprits.txt


之后再帮助编译器识别变量类型：
let myCompany: Dictionary<String, AnyObject> = 。。。。



+ [参考：](https://thatthinginswift.com/debug-long-compile-times-swift/)
+ [参考：](http://irace.me/swift-profiling)

