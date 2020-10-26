
---
title: Jenkins + Fastlane + 蒲公英 + iOS 自动化构建
date: 
tags:
---

## jenkins

### jenkins安装

```
brew install jenkins
```

### Jenkins 常用命令

```
 #启动jenkins
 brew services start jenkins
 #停止jenkins 
 brew services stop jenkins
 #重启jenkins
 brew services restart jenkins
 #直接启动jenkins
 jenkins
```

打开浏览器，输入localhost:8080,去相关路径找到密码复制进去即可

### 新建任务
点击 New Item，选择FreeStyle project,进入到任务配置界面

![jenkins_config](https://github.com/MaricleZhang/reasource/blob/master/create_jenkins_manager.png?raw=true)

### jenkins 配置

#### 安装Git 参数插件

1. Manage Jenkins -> Plugin Manager ->Git Parameter,安装完成重启jenkins，进入任务配置界面，选择“源码管理”中的Git选项,输入git地址
![git_source_config](https://github.com/MaricleZhang/reasource/blob/master/git_source_manage.png?raw=true)

2.  添加凭证(Credentials），输入git的账号密码
3.  配置git的选择参数（如果不设置该步骤，默认选择master分支执行）选中```This project is parameterized```按照下图输入,其中branch为自定义参数
![select_git_paramer](https://github.com/MaricleZhang/reasource/blob/master/select_git_paramer.png?raw=true)

在下面Source Code Management 中使用`${branch}` 如下图，点击Apply。
![set_git_brach](https://github.com/MaricleZhang/reasource/blob/master/set_git_brach.png?raw=true)

进入该任务界面,如下图说明配置成功

![build_with_parameters](https://github.com/MaricleZhang/reasource/blob/master/build_with_parameters.png?raw=true)

#### jenkins 执行脚本

##### 编译打包

```
export MallocNanoZone=0 
export LC_ALL=en_us.UTF-8 

cd ${JENKINS_HOME}/workspace/${JOB_NAME} #进入项目目录下

pod update --verbose  # Pod 依赖

fastlane development # fastlane 自定义命令
```

##### 上传到蒲公英

```
curl -F file=@/Users/zhangjian/.jenkins/workspace/JUApp-iOS/JUApp/build/JuApp.ipa -F buildInstallType=2 -F buildPassword=666 -F uKey=378617fba86912d68ddeda10fa9fbfc2 -F _api_key=1e9896673dc767184f8e3e2f8450a45f http://www.pgyer.com/apiv2/app/upload -X POST -H "enctype:multipart/form-data" > /tmp/upload_app.txt
url=$(cat /tmp/upload_app.txt|awk -F'"' '{print $(NF-1)}'|sed 's#\\##g')
echo "download_URL:<img src=$url>"

```


## Fastlane

### Fastlane 安装
选择xcode

 ```
 xcode-select --install
 ```
 
 安装Fastlane
 
 ```
 sudo gem install fastlane -NV
```
 
 ### 初始化Fastlane
 
 进入项目目录下执行
 
 ```
 fastlane init
 ```
 出现下面4个选项,选择4.自动执行`bundle update`需要几分钟时间
 
 ```
 1.  📸  Automate screenshots //自动截屏
 2. 👩‍✈️  Automate beta distribution to TestFlight //发布到TestFlight
 3. 🚀  Automate App Store distribution //App Store
 4. 🛠  Manual setup - manually setup your project to automate your tasks //自定义配置
 ```
 执行完毕后会出现fastlane文件夹，有两个文件Appfile和Fastfile。
 Appfile 配置apple id
 
```
 # app_identifier("[[APP_IDENTIFIER]]") # The bundle identifier of your app
# apple_id("[[APPLE_ID]]") # Your Apple email address

# For more information about the Appfile, see:
#     https://docs.fastlane.tools/advanced/#appfile

```
### 配置Fastlane 执行命令

```
default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"
  lane :custom_lane do
    # add actions here: https://docs.fastlane.tools/actions
  end

  desc "打development包"
  lane :development do |options|
    clear_derived_data(derived_data_path: "./DerivedData") # 清除本地编译缓存
    match(type: "development", force_for_new_devices: false, readonly: true) # fastlane match 管理证书
    gym(scheme: "FastlaneDemo",
      workspace: "FastlaneDemo.xcworkspace",
      configuration: "Debug",
      export_method: "development",#打包的类型
      output_directory: "./build",#生成的ipa路径
      output_name: "FastlaneDemo.ipa",#生成的ipa文件名
      silent: false,
      include_symbols: true,# 是否包含符号表
      derived_data_path: "./DerivedData",
      # clean:true,
      xcargs: "-UseNewBuildSystem=NO"
  )
  end
end
```

## 常见问题

### 出现 fastlane: command not found
这个情况一般是由于 jenkins 没有设置正确的 PATH，在命令行输入

```
echo $PATH
```

Manage Jenkins -> Configure System 选中Environment variables 在 key 中填写 PATH，在 value 中填写第一步中输出的结果 保存即可。如下图所示

![gobal_path](https://github.com/MaricleZhang/reasource/blob/master/gobal_path.png?raw=true)


 






