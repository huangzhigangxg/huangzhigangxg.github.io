---
layout: post

title: CocoaPods 组件化 实践

date: 2017-03-21 15:34:24.000000000 +09:00

---

## 1.创建WorkSpace包含主工程和业务模块FrameWorks

1. 创建一个文件夹 WorkSpace 
2. 通过 *Xcode -> File -> New -> WorkSpace* 创建一个 `WorkSpace.xcworkspace` 放在生成好的WorkSpace目录中
3. 创建一个主工程*main*放到在WorkSpace目录中
4. 创建一个Frameworks工程*Login*放到在WorkSpace目录中
5. 这时WorkSpace目录中有一个`WorkSpace.xcworkspace`和两个工程目录。
6. 打开`WorkSpace.xcworkspace`将两个工程的工程文件 *Main.xcodeproj*和*Login.xcodeproj*拽入`WorkSpace.xcworkspace`中。
7. 在这里不会发生仅导入工程文件丢失其他文件的问题，因为已经都将两个工程的文件放到了WorkSpace目录里了。
8. 添加编译依赖只需要在*Main*主工程的General的Embedded Binaries里选择业务模块工程生成的Framework就可以了。
9. 编译运行。

![](assets/images/WX20170322-153706@2x.png)

### 需要注意的地方

1. 在子模块Framework的接口要控制好权限.
	+ publie 修饰的class 和 func能被模块外调用 
	+ open 修饰的class不仅能能调用还能继承。
2. 因为是多个模块一起开发，所以每个子模块获取自己的资源时需要加上先获取到自己的*Bundle* 通过 *Bundle(for: SomeClassIn Framework.self)* 接受一个framework里的接口类，就能获取这个类所在的Bundle。

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

## 利用CocosPod在本地维护这些子模块

+ 在子模块中生成 xxxx.podspec 描述源码和资源的组织方式，和想一些项目信息等。
	+ cd Login && pod spec create Login

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
  s.source_files  = 'Login/Source/*.swift'
  s.resources = 'Login/Resources/*.{png,xib,storyboard,xcassets}'
```

+ 在主工程中生成Podfile描述需要导入哪些子模块
	+ cd Main && pod init
	+ 添加 Login 子模块，并指定本地Login.podspec路径
	+ pod install 

```
target 'Main' do
  use_frameworks!
  pod 'Login', :podspec => ‘../Login/Login.podspec'  
end

```
	
## 搭建私有cocoapods仓库

这个过程大概是这样，先创建私有Spec仓库，然后将自己的Pod工程push到Github上，把Pod的xxxx.spec传到私有Spec仓库中管理。将私有Spec仓库的地址soucre放到Podfile中，就可以使用像其他第三方库一样使用了。

1. 创建私有 Spec 仓库来管理私有 podspec 文件
	+ 在 Github上创建一个XGSpecs.git。
	+ 执行 `pod repo add XGSpecs https://github.com/callmebill/XGSpecs.git` 添加私有仓库到本地。
	
2. 检测自己Pod工程的podspec，并上传Github
	+ `pod lib lint` 检查podspec的有效性
	+ `git tag 0.0.2` & `git push --tags` 在Github上创建Login.git ,并上传Pod工程并打上tag 
	+ `pod repo push XGSpecs Login.podspec`将Pod工程的podspec文件传到私有仓库中管理。

3. 将私有Spec仓库的地址soucre放到Podfile中
4. 最后pod install

```
#先搜索私有仓库，再查找CocoaPods仓库。
source 'https://github.com/callmebill/XGSpecs.git' 
source 'https://github.com/CocoaPods/Specs.git'

target 'Main' do
  use_frameworks!
  pod 'Login' 
end
```

## 利用CocosPod将组件二进制化提高编译速度

1. 将生成好的Login.framework 放到工程路径下。ps:选用relese版本，因为Build active Architecture Only Debug 为YES。Relese包含了真正的你想要的架构版本 也就是 `Valid Architectures`和 `Architectures`的交集
2. 添加 `s.vendored_frameworks = 'Login/Login.framework'`
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
  # s.source_files  = 'Login/Source/*.swift'
  # s.resources = 'Login/Resources/*.{png,xib,storyboard,xcassets}'
  s.vendored_frameworks = 'Login/Login.framework'

```
下次Pod再被拉取时，就只有`Login/Login.framework`.没有资源文件和源代码。

![](assets/images/WX20170324-120831@2x.png)



## 发布项目到CocoaPods

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
