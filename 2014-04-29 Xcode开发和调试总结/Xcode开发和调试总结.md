# Xcode开发和调试总结

### 2014-04-29 15:01

Xcode是iOS开发主要的工具、IDE。关于Xcode的细枝末节，可以参考苹果的官方文档或者众多的说明。此文档主要涉及常用开发和调试注意事项，参考版本为Xcode 5.1.1。

## 目标设置

在此，我就不区分Project和Target了，这两方面有很多共同的设置，所以只需要了解需要设置哪些子项就可以了。

1. Deployment Target：设置支持的最低设备版本，这个根据代码的API支持情况而定
2. Base SDK：理论上应该设置为最新版本的SDK，以支持最高版本
3. Identity：设置Bundle Identifier（即AppId）、Version（版本号）、Build（编号）
4. Architecture：用于指定编译的目标架构，包括armv64、armv7、armv7s
5. App Icons、Launch Images：设置各种标准图标
6. Compiler：现在都默认为Apple LLVM，这是苹果专为C、C++、OC开发的编译器
7. Other Linker Flags：当引用第三方静态链接库时，需要加上-ObjC和-all_load
8. Info.plist File：指定项目的配置信息文件.plist，当然可以自己新建其他配置信息文件以供代码中使用
9. Prefix Header：前提是Precompile Prefix Header为Yes，指定了预编译头文件.pch
10. Search Paths：主要设置第三方引用的搜索路径，这就包括Framework（框架）、Header（头文件）、Library（静态链接库）
11. Linked Frameworks and Libraries：管理引用的框架和链接库
12. Code Signing Identity：主要设置本应用AppId产生的Debug、Distribution、Release三种模式的证书，来源为钥匙串
13. Provisioning Profile：主要设置本应用AppId和相应证书生成的概要配置文件，来源也是钥匙串，并且会决定Code Signing Identity中对应模式的备选证书
 
## 编译调试设置

1. 设置当前活动的模式：位于Xcode左上角的选项，包括设置目标项目和模拟器版本。例如需要先编译静态链接库或者改变启动项目。
2. 编辑模式：点击桌面菜单栏的Product->Scheme->Edit Scheme，可以设置更详细
3. 运行调试：可以直接点击“播放”按钮，也可以在菜单栏中的product子项中选择 run、Build For、Build、Clean等。
 
## Archive打包设置

1. 在编译调试设置的第二条显示界面中，选择左边的Archive，再设置Build Configuration项为Distribution
2. 将模拟器选择项设置为“iOS Device”（连接设备时，即为当前设备名称）
3. 在编译调试设置的第三条中，选择Archive（此前为灰色状态）
 
## 代码区自定义

1. 选择左上角的Xcode->Preferences进入Xcode设置界面。
2. Fonts&Colors：用于设置代码编辑区的背景和字体样式
3. Text Editing：用于显示行数、自动填充代码等等
4. Key Bindings：显示快捷键操作，也可自定义快捷键

 

## 模拟器使用简介

模拟器是用于仿真iPhone和iPad运行，显示App界面和功能。但是需要注意，模拟器并不等于真机运行，因为模拟器cpu采用的是i386架构，但是iOS真机采用的是armv架构；并且模拟器不支持远程推送之类的功能。

### 模拟器上的操作总结

Command + H：隐藏模拟器

Command + Q：推出模拟器

Command + S：截屏模拟器，存储到OS桌面

Command + L：锁屏

Command + ←：向左旋屏

Command + →：向右旋屏

Command + Shift + H：返回模拟器主界面，等于Home键

在iOS模拟器菜单栏，点击硬件->设备，可以切换模拟器设备

### 模拟器的iPhone SDK

/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs

上述路径，可以在Finder中Command+Shift+G搜索。

### 模拟器的沙盒路径

这是在当前用户账户的隐藏目录下，路径为

/Users/[USERNAME]/Library/Application Support/iPhone Simulator/7.1/Applications

此处的7.1为当前模拟器iOS版本，在Applications目录下，可以看到不同文件夹，代表不同的App。然后就可以看到App的沙盒目录：Documents、Library、tmp三个文件夹。

## 编译错误分析

编译错误种类太多，我只描述一下遇到的常见类型和重要错误。

常见错误，大多都是代码编写的问题，例如变量未实例化、对象引用计数为-1、向nil对象发送方法请求等。

### 比较重要的有：

1. Reference、link相关：即引用相关错误，多半是链接库或者头文件引用找不到，或者重复引用的问题。需要检查头文件引用，或者头文件搜索路径的配置。
2. 带有i386关键字的问题：多指编译目标架构不对应，i386架构只针对模拟器运行，但是真机编译，需要armv架构；或者是引用的静态链接库的编译架构有误。
3. 某些api被废弃：一般出现在更新了新版SDK后，需要找到对应api，然后替换为最新方法
4. 证书错误：这个返回到目标设置里去调整，或者需要去开发者中心重新配置生成
5. 预编译头文件.pch被修改：这个问题不大，clean以后重新编译即可