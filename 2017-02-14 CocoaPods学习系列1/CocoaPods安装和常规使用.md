# CocoaPods安装和常规使用

### 2017-02-14 17:13

CocoaPods是一个Github上的开源项目，目前已经成为iOS开发过程中标准的依赖库管理器，提供了一种对第三方类库简单优雅的集成和管理方案。

其工作原理，是将第三方类库统一管理到一个名为Pods的独立项目中，自动设置各种参数，然后让主项目通过只依赖该项目生成的.a静态链接库，就实现对所有第三方类库的依赖。

## gem包管理器

安装CocoaPods，通常使用终端指令的方式。目前也有CocoaPods客户端可以使用了，在此只记录终端指令的方式。

通过终端，需要使用Ruby语言的gem包管理器。Mac已经自带了Ruby环境，只需要检查一下gem包管理器的版本，输入：

```ruby -v```

或者 ```gem -v```

可以分别查看ruby和gem的版本。如果gem版本过低，可以使用升级指令：

```sudo gem update --system```

升级成功后会提示: ```Latest version currently installed. Aborting.```

升级gem后，查看一下ruby的软件源，输入：

```gem sources -l```

如果输出如下：

```
*** CURRENT SOURCES ***

https://rubygems.org/
```

则需要更换软件源，因为上述软件源是ruby默认的亚马逊软件源，未FQ则无法访问。

### 更换软件源

更换软件源，依次输入每行指令如下：

```
gem sources --remove https://rubygems.org/

gem sources -a https://gems.ruby-china.org/
```

再次查看软件源是否已经被正确修改了。

之前用的https://ruby.taobao.org/貌似已经无效了。

至此，gem设置完毕。

 

## 安装CocoaPods

只需要一行指令即可：

```sudo gem install cocoapods```

如果提示错误：

```
ERROR:  While executing gem ... (Errno::EPERM)

Operation not permitted - /usr/bin/xcodeproj
```

### 解决方案1：

```sudo gem install cocoapods -V```

如果依旧有错误，使用方案2

### 解决方案2：

```
sudo gem install -n /usr/local/bin cocoapods 

pod setup 
```

### 过程说明

pod setup在执行时，会输出```Setting up CocoaPods master repo```

安装成功后,你会看到:```Setup completed```

所有项目的Podspec文件都在```https://github.com/CocoaPods/Specs```。

第一次执行pod setup可能会等待特别长的时间，因为该操作实际上是将镜像索引specs克隆到```~/.cocoapods/repos/master```目录下。

目前的specs大概有二百多M，所以需要较长的时间。

### 加快pod setup速度的方案1

在执行pod setup前，输入：

pod repo remove master

pod repo add master https://gitcafe.com/akuandev/Specs.git 或者 pod repo add master https://git.oschina.net/akuandev/Specs.git

pod repo update

### 加快pod setup速度的方案2

但是我尝试后，依然会出现一些错误提示。

推荐另一种解决方案，执行：

pod repo remove master

git clone https://github.com/CocoaPods/Specs.git ~/.cocoapods/repos/master

pod repo update

其实就是将第二步，替换为直接用git将specs克隆到目标目录，仓库位置可以使用其他仓库替代。

最后执行pod setup，出现Setup completed即成功安装了CocoaPods。

## 使用CocoaPods添加第三方类库依赖

### 创建podfile文件

在终端中使用cd指令，cd到项目根目录下，即与xcodeproj工程文件同目录。

#### 1. 使用pod init指令

创建Podfile，可以使用```pod init```指令，生成模板内容，再自行修改。

#### 2. 使用vim Podfile指令

输入：

```vim Podfile```

该指令将创建一个Podfile文件，终端界面进入Podfile编辑状态。

### 配置Podfile文件

CocoaPods添加第三方库，需要先配置Podfile文件。

以添加AFNetworking库为例，输入如下内容：

```
source 'https://github.com/CocoaPods/Specs.git'

platform :ios, '8.0'


target 'xxx' do

pod 'AFNetworking', '~> 3.0'

end
```

以上内容，可以在AFNetworking的Github首页找到。target的名称需要修改为当前项目的target名称。

然后，点击ESC键，输入：

```:wq```

就可以保存并退出编辑状态。如果遇到无法退出，则输入：

```:wq!```

如果需要在终端中查看或编辑Podfile，再此输入：

```vi Podfile```

就可以了重复上述操作。

完成编辑后，输入：

```pod install```

等待执行完毕，即完成了对AFNetworking库的添加。

### 更多指令

输入：

```pod --help```

可以查看。

### 文件说明

进入项目根目录，将发现多了几个文件和文件夹。

1. Podfile记录了对第三方类库的依赖配置。
2. Podfile.lock文件会锁定当前各依赖库的版本，之后即使多次执行pod install也不会修改。只有执行pod update才会修改Podfile.lock。
3. Pods文件夹存放了第三方类库及引用配置

这些文件都应该始终被版本控制器管理。

## 常规操作

1. 打开工程项目，需要使用.xcworkspace文件
2. 可以在Xcode中直接更新Pods项目下的Podfile文件
3. 需要再次更新Podfile，只需要执行pod install或者update
4. 更新Podfile时候，若不需要更新Podspec索引，可以在上述指令后增加--no-repo-update
5. 可以在终端中搜索类库，例如：pod search SDWebImage