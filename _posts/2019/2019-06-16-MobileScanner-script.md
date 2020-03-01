---
layout: post

title: MoblieScanner 一站式扫描平台[脚本]

date: 2019-06-16 15:34:24.000000000 +09:00

---

![1](/assets/mobilescanner/mobilescanner_jenkins.png)

## 脚本语言基本实现原理
### 语言层面 JS python oc shell 对比
### 它们都怎么产生的
### 他们运行时什么原理？
### 都在什么场景下适宜使用
### 都有各自的什么特点
### 为什么要有这么多语言，都用c语言不香么

### 编译型语言与脚本语言 
每次执行脚本，虚拟机总要多出加载和链接的流程

完成模块的加载和链接；
将源代码翻译为PyCodeObject对象（这货就是字节码），并将其写入内存当中（方便CPU读取，起到加速程序运行的作用）；
从上述内存空间中读取指令并执行；
程序结束后，根据命令行调用情况（即运行程序的方式）决定是否将PyCodeObject写回硬盘当中（也就是直接复制到.pyc或.pyo文件中）；
之后若再次执行该脚本，则先检查本地是否有上述字节码文件。有则执行，否则重复上述步骤。



## 扫描功能

### 静态分析

![1](/assets/mobilescanner/someone.png)

#### 选用的是出自Facebook的 [Infer](https://fbinfer.com) 扫描工具、

#### 通过 xcodebuild 生成 xcodebuild.log 并用 xcpretty -r json-compilation-database 生成 compilation_db.json 文件用与infer接下来的分析, 它在build/reports目录下。

```
xcodebuild archive -archivePath ${myarchivePath} -workspace ${myworkspace}.xcworkspace -scheme ${myscheme} -UseModernBuildSystem=NO -configuration Release | tee xcodebuild.log  | xcpretty _0.2.8_ -r json-compilation-database
```

#### infer 解析 json-compilation-database 并设置扫描范围

```
infer 
--compilation-database build/reports/compilation_db.json 
--keep-going 								
--skip-analysis-in-path-skips-compilation //开启跳过
--skip-analysis-in-path "Pods/."		      //具体跳过哪些Pods
```

#### 将Bug分部到人,依赖以onetool开发模式，也就是clone远程git仓库的源码。有了git的信息，就能通过git blame命令拿到最后一个更新的人的作者，在python有git的增强，直接用Repo(repo_path)初始化后，就能拿到足够的信息

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



