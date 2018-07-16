# CocoaPods的进阶用法

### 2017-03-13 14:19

这篇文章，记录一下CocoaPods的进阶用法。

进阶用法主要体现在.podspec文件和Podfile的配置上。

## .podspec文件的进阶配置

以官方的一个.podspec文件示例细说：

```
Pod::Spec.new do |spec|
  spec.name         = 'Reachability'
  spec.version      = '3.1.0'
  spec.license      = { :type => 'BSD' }
  spec.homepage     = 'https://github.com/tonymillion/Reachability'
  spec.authors      = { 'Tony Million' => 'tonymillion@gmail.com' }
  spec.summary      = 'ARC and GCD Compatible Reachability Class for iOS and OS X.'
  spec.source       = { :git => 'https://github.com/tonymillion/Reachability.git', :tag => '3.1.0' }
  spec.module_name  = 'Rich'

  spec.ios.deployment_target  = '9.0'
  spec.osx.deployment_target  = '10.10'

  spec.source_files       = 'Reachability/common/*.swift'
  spec.ios.source_files   = 'Reachability/ios/*.swift', 'Reachability/extensions/*.swift'
  spec.osx.source_files   = 'Reachability/osx/*.swift'

  spec.framework      = 'SystemConfiguration'
  spec.ios.framework  = 'UIKit'
  spec.osx.framework  = 'AppKit'

  spec.dependency 'SomeOtherPod'
end
```
 
### 说明：
#### 1. |spec|中的spec命名可以任意修改，只需要保持后续选项的对象名称一致即可。

#### 2. version字段对应Podfile文件配置中版本号，source的tag字段对应SCM源代码的标签

#### 3. source字段指定了SCM源代码地址和版本，支持如下关键字组：

```
:git => :tag, :branch, :commit, :submodules

:svn => :folder, :tag, :revision

:hg => :revision

:http => :flatten, :type, :sha256, :sha1

:path
```
tag的写法可以如下：:tag => spec.version

建议如此，使得SCM源代码的标签和Podfile配置中版本号保持一致。

#### 4. source_files字段，表明类库包括的文件。

需要注意文件匹配的写法：

*表示文件名通配符

**表示文件夹递归匹配

?表示一个任意字符的匹配

[a-z]表示匹配集合中的一个字符

{h,m}表示两个字符中的任意一个

如下示例：

```
"JSONKit.?"    #=> ["JSONKit.h", "JSONKit.m"]
"*.[a-z][a-z]" #=> ["CHANGELOG.md", "README.md"]
"*.[^m]*"      #=> ["JSONKit.h"]
"*.{h,m}"      #=> ["JSONKit.h", "JSONKit.m"]
"*"            #=> ["CHANGELOG.md", "JSONKit.h", "JSONKit.m", "README.md"]
```

#### 5. ```public_header_files```和```private_header_files```分别表示公开和私有的头文件路径，便于集成时候自动配置引用路径，默认公开全部头文件

#### 6. ```vendored_frameworks```和```verdored_libraries```分别表示当前类库中存在的.framework和.a文件的路径，用于集成时候自动配置引用路径

#### 7. ```resource_bundles```字段，表明编译类库需要的资源bundles及其文件，如下：

```
spec.resource_bundles = {
    'MapBox' => ['MapView/Map/Resources/*.png'],
    'OtherResources' => ['MapView/Map/OtherResources/*.png']
  }
```

key表示bundle的名称，value表示内部资源文件。

不建议使用类似字段resources，可能造成资源名字冲突，并且会将指定资源直接拷贝到目标target中。

#### 8. exclude_files字段，表明不包括的文件

#### 9. frameworks和libaries字段表明类库引用的系统frameworks和libraries

需要注意的是，libraries的value需要省略前缀"lib"，并且不需要扩展名。例如需要引用libz.dylib，则如下写法：

```s.libraries = 'z'```
 

 

#### 10. ```compiler_flags```字段表明编译flags，例如-Objc

#### 11. ```pod_target_xcconfig```字段表明任意编译flag，如下：

```spec.pod_target_xcconfig = { 'OTHER_LDFLAGS' => '-lObjC' }```

#### 12. ```header_dir```字段表明存储公开头文件的文件夹名称

#### 13. dependency字段，表明依赖的其他类库及版本，多个类库分开书写

#### 14.```requires_arc```字段，表明是否启用ARC，默认为true，也可以指定部分目录或文件启用ARC，如下：

```
spec.requires_arc = 'Classes/Arc'
spec.requires_arc = ['Classes/*ARC.m', 'Classes/ARC.mm']
```

如果大多数文件启用ARC，个人文件不启用ARC，可以使用subspec字段：

```
Pod::Spec.new do |s|
  ...
  s.exclude_files = 'xxxx'

  s.subspec 'Core' do |sp|
    sp.requires_arc = false
    sp.source_files = 'xxxx'　　
  end

  s.subspec 'ObjectMapping' do |os|
  end
end
```

#### 15. subspec字段，定义一个子模块，将部分文件移入其中，可添加局部配置，'Core'和'ObjectMapping'就是子模块文件夹的名称

## Podfile文件的进阶配置

#### 1. pod指令，指定依赖库时候，无版本号表示最新版本，具体版本号将锁定版本，使用运算符可指定一个区间的版本；~> 0.1.2表示大于等于0.1.2并且小于0.2.0，此运算符对所给版本号最后一个部分有效。

#### 2. 如果需要使用本地仓库，可以如下配置：

```pod 'AFNetworking', :path => '~/Documents/AFNetworking'```

但需要注意，所给路径下应该存在.podspec文件 

#### 3. 如果podspec文件存在于git仓库中，即使没有最新提交对应的podspec文件，也可用下列方式直接集成对应位置的类库：

```
To use the master branch of the repository:
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git'


To use a different branch of the repository:
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :branch => 'dev'


To use a tag of the repository:
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :tag => '0.7.0'


Or specify a commit:
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :commit => '082f8319af'
```

特别是指定一个commit的方式，适合未发布标签版本的代码内测。 

#### 4. 如果podspec文件不存在于git仓库中，需要用如下方式访问：

```pod 'JSONKit', :podspec => 'https://example.com/JSONKit.podspec'```
 

#### 5. 如果想消除全部依赖类库的编译提醒，可使用inhibit_all_warnings!，如下配置：

```
platform :ios, '9.0'
inhibit_all_warnings!

target 'MyApp' do
  pod 'ObjectiveSugar', '~> 0.5'
  ...
end
```

也可以针对个别依赖库设置编译提醒的开关：

```
pod 'SSZipArchive', :inhibit_warnings => true
pod 'SSZipArchive', :inhibit_warnings => false
```