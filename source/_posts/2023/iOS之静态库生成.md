---
title: iOS之静态库生成
date: 2023-07-11 17:04:36
tags: [iOS]
category: [iOS]
---

最近在公司接手了主要项目，做了公司自己SDK库的开发，也大概总结了一些步骤，就此记录下，另外，先介绍下静态库：

```
关键特点：

编译时链接： 静态库在编译时被完全链接到应用程序中。这意味着库的代码被复制到应用程序的可执行文件中，使得应用程序在运行时不再依赖于原始的库文件。

独立性： 由于静态库在编译时被嵌入到应用程序中，应用程序在其他系统上运行时无需依赖外部库的存在。这增加了应用程序的独立性和可移植性。

多语言支持： 静态库通常可以用于多种编程语言，只要它们符合相应的二进制接口（ABI）。这使得开发者能够在不同的语言中共享和重用代码。

快速执行： 由于静态库的代码被直接嵌入到应用程序中，其执行速度可能比动态库稍快，因为无需在运行时进行额外的加载和链接操作。

使用场景：

模块化开发： 静态库促进了代码的模块化和重用，允许开发者将常用的功能打包成库，方便在不同项目中共享。

独立发布： 开发者可以将自己的功能或工具以静态库的形式发布，供其他开发者集成到他们的应用程序中。

性能优化： 静态库的使用可以在一定程度上提高应用程序的性能，尤其是对于小型项目或特定场景。

```

静态库搭建，并且上传到pod，主要分为以下几步：
1.创建项目；
2.编写代码；
3.构建静态库；
4.查找库文件；
5.创建podspec文件；
6.提交到GitHub
7.发布到CocoaPods

### 1.创建项目

首先，我们要打开Xcode，并创建了一个新的项目。在项目模板选择界面，注意要选择了"Framework & Library"，然后点击"Static Library"。这样就创建了一个静态库的项目。

### 2.编写代码

接下来就是正常的编写自己项目的代码，并且调试，注意这里既然时一个库的话，就要好好注意下规范，可复用性，代码安全等东西。


### 3.构建静态库

完成代码编写后，在xcode中，选择项目的 scheme，并将设备设置为“Generic iOS Device”。然后，点击 Xcode 的 Product -> Build 功能来构建静态库。

### 4.查找库文件

构建成功后，前往Xcode的DerivedData目录中查找生成的静态库文件。通常情况下，它们位于`~/Library/Developer/Xcode/DerivedData/YourLibrary-xxxx/Build/Products/Debug-iphoneos/YourLibrary.a`路径下。

### 5.创建podspec文件

接下来，在项目的根目录创建了一个`.podspec`文件。这个文件包含了有关我的库的信息，例如名称、版本、作者等，并指定了静态库文件的路径。

```ruby
Pod::Spec.new do |s|
  s.name         = 'YourLibrary'
  s.version      = '1.0.0'
  s.summary      = 'A brief description of YourLibrary.'
  s.homepage     = 'https://github.com/yourusername/YourLibrary'
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.author       = { 'Your Name' => 'youremail@example.com' }
  s.source       = { :git => 'https://github.com/yourusername/YourLibrary.git', :tag => s.version.to_s }

  s.ios.deployment_target = '12.0'

  s.source_files  = 'YourLibrary/**/*.swift' # 调整以符合你的文件结构
  s.public_header_files = 'YourLibrary/**/*.h' # 调整以符合你的文件结构

  s.frameworks = 'UIKit' # 如果你的库需要，添加框架

  # 这里可以添加其他规格，比如依赖项

end
```

### 6.提交到仓库

将项目提交到了GitHub或类似的代码仓库，要检查下`.podspec`文件也包含在其中。

### 7.发布到CocoaPods

最后，运行以下命令将我的库发布到了CocoaPods：

```bash
pod trunk register your@email.com 'Your Name' --description='Your description'
pod trunk push YourLibrary.podspec
```

到这里为止，其他人就可以通过在他们的项目中添加`pod 'YourLibrary'`来下载和使用我们的第三方库了。

大概就是这些，整体梳理下来的大概流程，具体实践中肯定还会遇到很多问题，就要自己一个个解决了。