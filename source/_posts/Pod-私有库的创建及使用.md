---
title: Pod 私有库的创建及使用
date: 2018.02.12 17:57
tags:
---
## 前言
CocoaPods是iOS项目的依赖管理工具，使用它可以方便的管理和更新项目中所使用到的第三方库，我们一般用的是公有库，代码存放在Github上。个人或公司在开发过程中，会积累很多可以复用的代码包，有些我们不想开源，又想像开源库一样在CocoaPods中管理它们，那么通过私有库来管理就很必要。
<!-- more -->
## 原理
在创建私有库之前，我们有必要理解私有库的原理，
![podfile、索引库和代码库关系图.png](https://github.com/MaricleZhang/reasource/blob/master/pod_spec_icon.png?raw=true)

Podfile:工程的依赖描述文件。
Spec:  该仓库存放索引描述文件.podspec，CocoaPods通过该文件对你真正存储代码工程的 Git 仓库进行索引与下载,下面会有对podspec的详细介绍。通过spec库来控制pod私有库的版本。

Lib : 你上传到远程Git仓库的代码工程，将来用于开源共享或则私有

CocoaPods 根据podfile中的库的名称，通过spec索引库，来找到真正的代码库。这里有同学可能要问为什么需要两个仓库呢，我的理解是为了快速检索和版本控制。如果没有spec索引库，我们每次pod update的时候都需要在GitHub所有的代码库中检索。显然效率很低。

## Pod私有库的创建步骤
1. 创建两个仓库：索引库和代码仓库。
2. 创建并设置本地私有Spec Repo。
3. 创建Pod所对应的podspec文件。
4. 本地测试配置好的podspec文件。
5.向私有的Spec Repo中提交podspec。

### 创建两个仓库：索引库和代码仓库
在github上创建索引库 **[ZJMaricleSpec](https://github.com/MaricleZhang/ZJMaricleSpec)**和代码库**[ZJMaricle](https://github.com/MaricleZhang/ZJMaricle)**

### 创建并设置本地私有Spec Repo
两个仓库创建完成后在Terminal中执行如下命令
```
 # pod repo add [Private Repo Name] [GitHub HTTPS clone URL]
$ pod repo add ZJMaricleSpec https://github.com/MaricleZhang/ZJMaricleSpec.git
```
需要特别注意的是这里的URL是Spec索引库的地址，如果写成代码库的地址就会出现
```
[!] An unexpected version directory Assets was encountered for the /Users/zhangjian/.cocoapods/repos/mariclezhang/YTVestSDK Pod in the YTVestSDK repository.
```
执行完成后
```
$ cd ~/.cocoapods/repos
```
如果存在`ZJMaricleSpec`则说明本地`repo Spec` 创建成功
### 创建Pod所对应的podspec文件
  cd 到要创建项目的目录然后执行
```
$ pod lib create ZJMaricle
```
然后会问一些问题
```
What language do you want to use?? [ Swift / ObjC ]
 > Objc
//这里特别说明一下，最好带上demo方便我们以后的代码调试
Would you like to include a demo application with your library? [ Yes / No ]
 > Yes

Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None

Would you like to do view based testing? [ Yes / No ]
 > Yes

What is your class prefix?
 > ZJ
```
问完这4个问题他会自动执行pod install命令创建项目并生成依赖。
下面是生成的文件目录及介绍
```
$ tree ZJMaricle -L 2
ZJMaricle
├── Example                                            #demo 
│   ├── Podfile                                        #demo 的依赖描述文件
│   ├── Podfile.lock
│   ├── Pods                                           #demo 的依赖文件
│   ├── Tests                                           
│   ├── ZJMaricle
│   ├── ZJMaricle.xcodeproj
│   └── ZJMaricle.xcworkspace
├── LICENSE
├── README.md
├── ZJMaricle
│   ├── Assets                                         #资源文件
│   └── Classes                                        #类文件
├── ZJMaricle.podspec                                  #podspec pod库的描述文件
└── _Pods.xcodeproj -> Example/Pods/Pods.xcodeproj

10 directories, 5 files
```
接下来把库文件添加到`ZJMaricle/Classes`文件夹下，终端进入`ZJMaricle/Example`下，执行`pod update`命令,完成后打开项目工程可以看到，刚刚添加的组件已经在Pods子工程下Development Pods/ZJMaricle中了。这里需要注意的是：每次向Pods 中添加文件或者podspec中的版本升级都要执行`pod update`命令。

配置podspec文件
```
#
# Be sure to run `pod lib lint ZJMaricle.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'ZJMaricle'                              #名称
  s.version          = '0.1.0'                                  #版本号
  s.summary          = 'description of ZJMaricle.'              #库的描述

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/MaricleZhang/ZJMaricle.git' #主页,这里要填写可以访问到的地址，不然验证不通过
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { '929006968@qq.com' => 'jian.zhang@yuntu-inc.com' }
  s.source           = { :git => 'https://github.com/MaricleZhang/ZJMaricle.git', :tag => s.version.to_s }  #项目地址，这里不支持ssh的地址，验证不通过，只支持HTTP和HTTPS，最好使用HTTPS
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '8.0'

  s.source_files = 'ZJMaricle/Classes/**/*'                        #库文件
  
  # s.resource_bundles = {                                         #库资源文件
  #   'ZJMaricle' => ['ZJMaricle/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end

```
### 本地测试配置好的podspec文件。
编辑`.podspec`完成后本地校验是否合格，这里有必要解释一下`pod lib lint`校验本地`.podspec`文件和pod库中的文件的合法性，如果不合法，则需要修改再校验，直到校验通过。如果有警告则需要使用`pod lib lint --allow-warnings`命令通过校验。
```
$ pod lib lint
```
当出现下面情况时说明本地校验成功
```
 -> ZJMaricle (0.1.0)

ZJMaricle passed validation.
```
提交代码，我这里使用的`sourcetree`,本地代码库与远程库建立连接，这里就不在赘述了。并且给远程仓库打上tag，需要注意的是这里的tag就是创建私有库的版本号，需与 `.podspec`文件中的版本号保持一致。重要的事情说三遍：本地校验成功后在打tag，本地校验成功后在打tag，本地校验成功后在打tag。
![给GitHub仓库打上tag.png](https://github.com/MaricleZhang/reasource/blob/master/source_tree_tag.png?raw=true)
### 向私有的Spec Repo中提交podspec
确认本地校验成功，并且打上tag后，终端进入到`ZJMaricle.podspec`同一目录下执行
```
$ pod repo push ZJMaricleSpec ZJMaricle.podspec  #前面是本地Repo名字 后面是podspec名字
```
出现
```
$ pod repo push ZJMaricleSpec ZJMaricle.podspec

Validating spec
 -> ZJMaricle (0.1.0)

Updating the `ZJMaricleSpec' repo

Your configuration specifies to merge with the ref 'refs/heads/master'
from the remote, but no such ref was fetched.

Adding the spec to the `ZJMaricleSpec' repo

 - [Add] ZJMaricle (0.1.0)

Pushing the `ZJMaricleSpec' repo
```
到索引库Spec Repo远端仓库，会发现有了一次提交，说明podspec已经push上去了，并且索引库已经与代码库建立了有效连接。再到本地的Sepc repo中
```
$ cd ~/.cocoapods/repos/ZJMaricleSpec
```
会发现目录结构变成
```
ZJMaricle
└── 0.1.0
    └── ZJMaricle.podspec
```
到这里pod私有库的创建已经完成，怎么样检验我的pod私有库有没有建立成功呢。执行`pod search ZJMaricle`出现说明创建成功。
```
-> ZJMaricle (0.1.0)
   description of ZJMaricle.
   pod 'ZJMaricle', '~> 0.1.0'
   - Homepage: https://github.com/MaricleZhang/ZJMaricle.git
   - Source:   https://github.com/MaricleZhang/ZJMaricle.git
   - Versions: 0.1.0 [ZJMaricleSpec repo]
(END)
```
这里讨论的是pod私有库，如果建立共有库的话，请使用 `trunk`工具,详细请参考[官方文档](http://guides.cocoapods.org/making/getting-setup-with-trunk.html)。

## Pod 私有库的使用
### 1. 私有库的引用
当我们在Example下执行`pod install` 的时候会出现`[!] Unable to find a specification for ZJMaricle `,Cocoapods并不能找到对应的私有库。这是因为默认情况下会在公有库https://github.com/CocoaPods/Specs.git 中查询，我们的私有库并不在这里，所以当然查询不到。下面是解决的办法，在podfile的顶部添加
```
source 'https://github.com/CocoaPods/Specs.git'  #官方仓库的地址
source 'https://github.com/MaricleZhang/ZJMaricleSpec.git'  #我们自己的私有spec仓库的地址
```
这里添加的一定为Spec库的地址，如果写成代码库则会出现
```
[!] An unexpected version directory Assets was encountered for the /Users/.cocoapods/repos/mariclezhang/ZJMaricleSpec Pod in the ZJMaricleSpec repository.
```
## 2.Pod库的校验
#### pod lib lint 和 pod spec lint 命令的区别
` pod lib lint`是只从本地验证你的pod能否通过验证
 `pod spec lint`是从本地和远程验证你的pod能否通过验证
在提交代码之前打tag之前，使用pod lib lint 验证一般不会出现问题，如果出现问题需要验证远程的代码则使用`pod spec lint`来验证。
本地校验`pod lib lint`如果出现
```
[!] ZJMaricle did not pass validation, due to 1 warning (but you can use `--allow-warnings` to ignore it).
```
在代码中警告无法清除时，在提提交代码时使用
```
pod repo push ZJMaricle ZJMaricleSpec.podspec --allow-warnings
```
参考文档：
[使用Cocoapods创建私有podspec](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)
[Cocoapods使用私有库中遇到的坑](http://www.jianshu.com/p/1e5927eeb341)
[Cocoapods 应用第二部分-私有库相关](http://www.cocoachina.com/ios/20150930/13471.html)




