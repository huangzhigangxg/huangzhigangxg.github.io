---
layout: post

title: 自动化发包fastlane

date: 2016-11-18 15:32:24.000000000 +09:00

---
## 自动化发包fastlane

从Archive开始到商店上架，所有的操作可以由fastlane帮你完成，简明来说要配置一个Fastfile文本文件，这里面定义的很多命令集。 

常用的命令集可以命名为 比如 uploadTestfight 用于上传Testfight内部分发测试，AppStore 用于直接发布app和更新商店内容。

命令集中 有很多命令 他们有 最基本的 

+ gym：可以编译代码  
+ pilot：可以上传Testfight并且管理测试成员
+ scan：每次发包前可以自动跑一遍所有Test
+ increment_build_number：增加build 版本号
甚至是其他的公司开发的工具Bugly，等等也有对fastlane的支持，有写好的
+ action可以直接放在fastlane/action下就能直接使用
等等很多很多的命令甚至是脚本，都可以放进命令集中，帮你用一行代码完成这些所有的事。

+ TODO：test 测试viewModel 接口
+ TODO：Bugly.sh 获取的share URL 直接发到测试同学的微信群里。标明修改后待测试的bug号，

## 最后Fastfile文件
```
  desc "Runs all the tests"
  lane :test do
    scan
  end

  desc "Submit a new Beta Build to Apple TestFlight"
  lane :TestFlight do
    increment_build_number
    gym(scheme: "LeVR") # Build your app - more options available
    pilot
  end

  desc "Submit a new Beta Build to Bugly”
  lane :Bugly do
    # increment_build_number
    gym(scheme: “LeVR” ,
	configuration:“AdHoc”,
	export_method: 'ad-hoc')
    upload_app_to_bugly(
        file_path:"**********.ipa”,
        app_key:"**********",
        app_id:"**********",
        pid:”2”,
        title:”**********“,
        desc:"**********",
        secret:”1”,
        users:"",
        password:"",
        download_limit:1000
    )

  end
```


