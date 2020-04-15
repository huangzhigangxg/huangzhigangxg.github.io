---
layout: post

title: MoblieScanner 一站式扫描平台[脚本]

date: 2019-06-16 15:34:24.000000000 +09:00

---

![1](/assets/mobilescanner/mobilescanner_jenkins.png)

MoblieScanner 是企业内部针对移动端定制的扫描平台，支持iOS/Andriod端，有静态扫描，应用权限扫描，多语言扫描，配置开关扫描等，由Jenkins执行脚本将分析后的数据上传后台服务并展示到前端，前端可以出发扫描任务，应用于测试接入环节。 

下面内容是Jenkins执行脚本的部分

## 扫描功能

### 静态分析

![1](/assets/mobilescanner/someone.png)

选用的是出自Facebook的 [Infer](https://fbinfer.com) 静态扫描工具

通过 xcodebuild 生成 xcodebuild.log 并用 xcpretty -r json-compilation-database 生成 compilation_db.json 文件用于infer接下来的分析, 它在build/reports目录下。

```
xcodebuild archive -archivePath ${myarchivePath} -workspace ${myworkspace}.xcworkspace -scheme ${myscheme} -UseModernBuildSystem=NO -configuration Release | tee xcodebuild.log  | xcpretty _0.2.8_ -r json-compilation-database
```

infer 解析 json-compilation-database 并设置扫描范围

```
infer 
--compilation-database build/reports/compilation_db.json 
--keep-going 								
--skip-analysis-in-path-skips-compilation //开启跳过
--skip-analysis-in-path "Pods/."		      //具体跳过哪些Pods
```

将Bug分派到人,依赖以onetool开发模式，也就是clone远程git仓库的源码。有了git的信息，就能通过git blame命令拿到最后一个更新的人的作者，在python有git的增强，直接用Repo(repo_path)初始化后，就能拿到足够的信息

```
def editorFrom(self,project_path,file_path,line_num):
	paths = file_path.strip('/').split('/')
	paths.reverse()
	repo_path = project_path + '/' + paths.pop()
	repo = Repo(repo_path)
	file_absolute_path = project_path + '/' + file_path
	cur = 0
	for commit, lines in repo.blame('HEAD', file_absolute_path):
		if cur <= line_num <= (cur + len(lines)):
			return commit.author.name
		cur += len(lines)
	return ''
```

[Infer Manuals](https://fbinfer.com/docs/man-pages.html)
 

### 权限对比

![1](/assets/mobilescanner/quanxianduibi.png)

## 脚本语言基本实现原理

### 编程语言的发展史 

抽象，这是整个计算机发展史，以及编程思想的核心，负责帮我们剔除不需要的部分，看清重要的，并提取出来，形成框架。

+ 面向机器
+ 面向过程<顺序，分支，循环>
+ 面向对象<封装﹑继承﹑多态>OOP
+ 面向协议<协议，组合, 泛型>POP

### 动态类型与静态类型，强类型与弱类型语言

+ 强类型：偏向于不容忍隐式类型转换。譬如说haskell的int就不能变成double
+ 弱类型：偏向于容忍隐式类型转换。譬如说C语言的int可以变成double
+ 静态类型：编译的时候就知道每一个变量的类型，因为类型错误而不能做的事情是语法错误。
+ 动态类型：编译的时候不知道每一个变量的类型，因为类型错误而不能做的事情是运行时错误。譬如说你不能对一个数字a写a[10]当数组用。

Swift 是静态强类型语言，OC 是动态强类型语言

### 编译型语言 与 解释语言 

每次执行脚本，虚拟机总要多出加载和链接的流程

完成模块的加载和链接；
将源代码翻译为PyCodeObject对象（这货就是字节码），并将其写入内存当中（方便CPU读取，起到加速程序运行的作用）；
从上述内存空间中读取指令并执行；
程序结束后，根据命令行调用情况（即运行程序的方式）决定是否将PyCodeObject写回硬盘当中（也就是直接复制到.pyc或.pyo文件中）；
之后若再次执行该脚本，则先检查本地是否有上述字节码文件。有则执行，否则重复上述步骤。

Java JS Python Go Swift C#
解释性语言，是在运行的时候将程序翻译成机器语言，在程序运行的前一刻还只有源代码没有可执行文件，程序每执行到源代码的某一条指令，就会有一个称为解释程序的外壳程序将源代码转换成二进制代码以供执行，也就是说，要不断地解释、执行、解释、执行、解释、执行.....。所以运行速度相对于C和C plus plus 慢，解释性语言的程序不需要编译，省了道工序，解释性语言在运行程序的时候才翻译。


### 语言运行时是什么？

C语言的时候 printf malloc stcpy之类的函数都是系统实现好的，
C++的时候，出现了函数库形式的运行时，这时runtime还是薄薄的一层
之后有个python等动态脚本语言，他们的runtime进阶了，不仅仅是标准函数库，还增加了GC，解释脚本等等
再后来有了Java，Go之类的静态类型高级语言，runtime 管理的事情就更多了，JIT编译等，这里用的是编译而不是解释，就说明是一次性编译后保存下来反复使用，而解释却是反复编译反复使用。nodejs用的V8就采用即时编译技术（JIT），直接将JavaScript代码编译成本地平台的机器码

runtime 就是一个语言实现的基础, 就好像一个人类最基本的心跳, 呼吸技能一样. runtime 和 库 的区别, 类似于 人类本身 与 人类后天增加的装备 的区别。
runtime 一般和 compile time 相对，
他们在时间上，分别代表运行期和编译期两个时期；
在代码上，runtime 代表程序能正常运行所必需的基础代码。
对于解释型语言，它的解释器就是 runtime；
对于编译型语言，它的 runtime 可以理解为标准库和系统库中不可或缺的那一部分。
总之， runtime 的意思大概就是 「运行期所必需的东西」。

[Zombie110year](https://www.zhihu.com/question/20607178/answer/572519795)



