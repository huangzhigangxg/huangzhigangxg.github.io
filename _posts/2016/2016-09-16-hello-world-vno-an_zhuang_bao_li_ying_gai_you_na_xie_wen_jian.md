---
layout: post

title: 安装包里应该有哪些文件

date: 2016-09-16 15:32:24.000000000 +09:00

---

今天，减小了乐视VR安装包（ipa）太大问题，发现从Unity导出的Xcode工程，有三个文件夹 Classes Libraries Data ，其中 Classes Libraries 导入时应选择Create groups 作为编译文件 会自动添加到 Compile Sources和Link Binary With Libraries 里，而导入Data时应选择Create folder references 做为资源文件自动添加到Copy Bundle Resources里。

而Data在以前的版本里导入方式是Create groups 作为编译文件，导致Compile Sources里添加了Data，最后编译的ipa中就会有两份Data，造成ipa体积的增加 大约37M。

还需要注意的是 Classes 作为编译文件被添加到Compile Sources，这里面里可能有各种执行文件 比如 .m .cpp .mm 编译后变成可执行文件，是不会出现在ipa里的。

Libraries 里静态库libiPhone.a 会被加到Link Binary With Libraries 里，这里面有一些第三方的或者苹果原生的FrameWork，在编译时用于连接，同样编译后编程可执行程序，不会出现在ipa里。

ipa里面所有的 除了最后编译的可执行程序，最主要的就是 资源文件：png xib bundle Assets.car 语言包 info.plist

再检查ipa时 可查看是否包含不必要的 东西。减小ipa的大小