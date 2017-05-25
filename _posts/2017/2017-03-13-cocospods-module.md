---
layout: post

title: CocoaPods 组件化 实践

date: 2017-03-21 15:34:24.000000000 +09:00

---

## 1.创建WorkSpace包含主工程和业务模块FrameWorks
> 在没有CocoaPods情况下，用Xcode管理多个project的方式。

1. 创建一个文件夹 WorkSpace 
2. 通过 `Xcode -> File -> New -> WorkSpace` 创建一个 `WorkSpace.xcworkspace` 放在生成好的WorkSpace目录中
3. 创建一个主工程`main`放到在WorkSpace目录中
4. 创建一个Frameworks工程`Login`放到在WorkSpace目录中
5. 这时WorkSpace目录中有一个`WorkSpace.xcworkspace`和两个工程目录。
6. 打开`WorkSpace.xcworkspace`将两个工程的工程文件 `Main.xcodeproj`和`Login.xcodeproj`拽入`WorkSpace.xcworkspace`中。
7. 在这里不会发生仅导入工程文件丢失其他文件的问题，因为已经都将两个工程的文件放到了WorkSpace目录里了。
8. 添加编译依赖只需要在`Main`主工程的General的Embedded Binaries里选择业务模块工程生成的Framework就可以了。
9. 编译运行。

![](/assets/images/WX20170322-153706@2x.png)



## 2.利用CocosPod在本地维护这些子模块
> 现在本地尝试用CocosPod管理`Framework`，确定无误再传到自己的Github，让团队里其他的小伙伴可以线上获取`Framework`。

+ 在`Framework`工程中生成Pod用来管理描述`Framework`源码和资源的组织方式的描述文件 xxxx.podspec。之后其他工程再需要这个`Framework`时只要找到这个描述文件就知道如何导入这里面的源码了。
	+ cd Login //进入`Framework`工程
	+ pod spec create Login //生成`Framework`的描述文件

```
Pod::Spec.new do |s|
  s.name         = "Login"
  s.version      = "0.0.2"
  s.summary      = "A short description of Login."
  s.description  = "A description for Login Framework"
 s.homepage     = "http://EXAMPLE/Login"
  s.license      = "MIT"
  s.author             = { "Xiao Gang" => "huangzhigang1024@qq.com" }
  s.source       = { :git => "https://github.com/callmebill/Login.git", :tag => "#{s.version}" }
  s.source_files  = 'Login/Source/`.swift'
  s.resources = 'Login/Resources/`.{png,xib,storyboard,xcassets}'
end
```

+ 在主工程中生成Podfile描述需要导入哪些`Framework`
	+ cd Main && pod init //生成Pod的依赖管理文件
	+ 添加 Login 子模块，并指定本地Login.podspec路径
	+ pod install //安装依赖

```
target 'Main' do
  use_frameworks!
  pod 'Login', :podspec => ‘../Login/Login.podspec'  
end

```
	
## 3.搭建私有cocoapods仓库
> 这一步做的是，将描述文件`Login.podspec`和`Framework`源码都上传到Github，让团队里其他的小伙伴可以线上获取`Framework`。

1. 在Github上创建`Framework`项目仓库 `Login.git`
2. 在Github上创建管理`Login.podspec`描述文件的私有仓库 `XGSpecs.git`
3. 	`cd Login` //进入`Framework`工程
4. `pod lib lint` //先检查podspec的有效性,并处理所有错误和警告
5. `git tag 0.0.2` & `git push --tags` //podspec验证无误后,上传`Framework`工程到Github并打上tag 
6. `pod repo add XGSpecs https://github.com/callmebill/XGSpecs.git` //添加刚刚创建的私有仓库到本地。
7. `pod repo push XGSpecs Login.podspec`//将`Framework`工程的podspec文件传到私有仓库中管理。
8. 将私有Spec仓库的地址soucre放到Podfile中
9. 最后pod install

```
#先搜索私有仓库，再查找CocoaPods仓库。
source 'https://github.com/callmebill/XGSpecs.git' 
source 'https://github.com/CocoaPods/Specs.git'

target 'Main' do
  use_frameworks!
  pod 'Login' 
end
```
![](/assets/images/WX20170413-134608@2x.png)
![](/assets/images/WX20170413-134700@2x.png)
![](/assets/images/WX20170413-134453@2x.png)


## 4.利用CocosPod将组件二进制化提高编译速度

1. 将生成好的Login.framework 放到工程路径下。ps:选用relese版本，因为Build active Architecture Only Debug 为YES。Relese包含了真正的你想要的架构版本 也就是 `Valid Architectures`和 `Architectures`的交集
2. 添加 `s.vendored_frameworks = 'Login/Login.framework'`到`Login.podspec `中
3. 因为Login.framework已经包含了资源，所以去掉s.resources
4. 因为不需要源代码，所以去掉s.source_files

注意：

1. 每次修改Pod工程 一定要打上tag！！！CocoaPods是通过tag获取git版本的。
2. 选用稳定的CocoaPods 1.1.1版本。之前用的 CocoaPods 1.2.1.beta.1 有缺少xcscheme的bug。
3. `$ lipo –create /Debug-iphoneos/Someframework.framwork/Someframework Debug-iphonesimulator/Someframework.framwork/Someframework –output Someframework`
4. 在用pod lib lint检测和pod install安装时，加上 --verbose 命令查看具体过程,发现问题。
5. 必要时删掉主工程的Pod文件，并`pod cache clean --all`重新尝试 `pod install`
6. 不要忘记将修改后的`Login.podspec`提交到私有XGSpecs.git

```
Pod::Spec.new do |s|
  s.name         = "Login"
  s.version      = "0.0.3"
  s.summary      = "A short description of Login."
  s.description  = "A description for Login Framework"
 s.homepage     = "http://EXAMPLE/Login"
  s.license      = "MIT"
  s.author             = { "Xiao Gang" => "huangzhigang1024@qq.com" }
  s.source       = { :git => "https://github.com/callmebill/Login.git", :tag => "#{s.version}" }
  # s.source_files  = 'Login/Source/`.swift'
  # s.resources = 'Login/Resources/`.{png,xib,storyboard,xcassets}'
  s.vendored_frameworks = 'Login/Login.framework'
end
```
下次Pod再被拉取时，就只有`Login/Login.framework`.没有资源文件和源代码。

![](/assets/images/WX20170324-120831@2x.png)

## 5.发布项目到CocoaPods

+ 注册trunk，不是任何人都能推送，因为cocoapods依赖trunk服务器管理，所以需要通过trunk推送自己的podspec（cocoapods官网）
+ 命令：pod trunk register EMAIL [NAME]
+ NAME: 表示NAME可有可无`
+ pod trunk register XXXXXXX@qq.com usernames
+ 验证成功后，点击邮箱就好了，打开会有点慢.
+ 推送自己的podspec到cocoapods的索引库
+ 命令：pod trunk push HttpManager.podspec --allow-warnings
+ 注意：podspec文件中的s.version版本号要跟最新Tag一致
+ 注意：podspec文件中的s.source仓库地址也不能写错

+ [教你如何从0到1实现组件化架构](http://www.jianshu.com/p/7b4667cde80b)
+ [iOS App组件化开发实践](http://www.yiqixiabigao.com/yin-ke-kong-gu-iosman-man-zu-jian-hua-kai-fa-zhi-lu/)
+ [iOS CocoaPods组件平滑二进制化解决方案](http://www.yiqixiabigao.com/ios-cocoapodszu-jian-ping-hua-er-jin-zhi-hua-fang-an-ji-xiang-xi-jiao-cheng/)
+ [蘑菇街组件化](http://ios.jobbole.com/89259/)
+ [美团组件化的二进制方案](http://www.cocoachina.com/ios/20170427/19136.html)