---
layout: post

title: 一站式扫描平台[后端]

date: 2019-06-16 15:34:24.000000000 +09:00

---


## 框架选择方案

方案 [iOSSnapshotTestCase](https://github.com/uber/ios-snapshot-test-case/) + [Specta](https://github.com/specta/specta) + [Expecta](https://github.com/specta/expecta)

	1. iOSSnapshotTestCase 以像素为单元做截图对比
	2. Specta 一个 OC 的 TDD/BDD 测试框架 ,让测试代码更易读
	3. Expecta 验证框架,还有 Expecta+Snapshots 支持截图对比.
 
为了我更好的观察截图对比结果, 需要给 Xcode 安装一个收集截图对比结果的插件 [Snapshots](https://github.com/orta/snapshots) 可以通过 [Alcatraz](http://alcatraz.io) 安装

## 备忘

1. 苹果出了Xcode8之后，就加了签名让之前的自定义插件无法继续的安装使用. 需要给 Xcode 重新签名, 最好留两个. 一个是直接安装的 用于打包. 另外一个重新签过名的用于使用插件开发.
2. 这种方法依然不是很稳定. 不要装太多插件.
3. Podfile 文件中 UnitTestTarget 可以与 主工程的 Target 并列.
4. 在生成的 Pods-XXXXXUnitTest.debug.xcconfig 中
	1. OTHER_CFLAGS 后面要链接 你将要测试的库,自己的主工程.
	2. OTHER_LDFLAGS 后面要链接 测试辅助框架, 
	3. 如果编译出错,比如符号冲突,重定义可以整理OTHER_LDFLAGS到最简.

```
<Podfile.h>

target 'XXXXX' do
    pod 'WhatYouNeed'
end

target 'XXXXXUnitTest' do
    pod 'Specta'
    pod 'Expecta+Snapshots'
    pod 'OCMock'
end
```
----------------------------------------------------
```
<Pods-XXXXXUnitTest.debug.xcconfig>

OTHER_CFLAGS = $(inherited) -isystem "${PODS_ROOT}/Headers/Public" -isystem "${PODS_ROOT}/Headers/Public/AFNetworking" -isystem

OTHER_LDFLAGS = $(inherited) -ObjC -l"Specta"  -l"Expecta" -l"Expecta+Snapshots" -l"FBSnapshotTestCase" -l"OCMock"

```
## 参考

1. [截图测试](https://objccn.io/issue-15-7/)
2. [截图测试框架 iOSSnapshotTestCase ](https://github.com/uber/ios-snapshot-test-case/)
3. [Xcode8以上 安装插件](http://www.cnblogs.com/jys509/p/6290416.html)

