# CocoaPods错误集锦

### 2017-03-13 14:50

这篇文章记录使用CocoaPods过程中遇到的一些错误。

## error：include of non-modular header inside framework module

在自定义类库中，引用了pop类库的POPAnimatableProperty.h头文件，配置好podspec文件后，执行pod lib lint时候，出现上述错误提示。

解决方法：

添加```--use-libraries```

最后执行：

```pod lib lint --use-libraries```


## error: 大量警告warnings

编译类库，难免存在警告，最好先解决警告，再编译检查。但是，如果需要忽略编译警告，则要添加额外设置。

解决方法：

添加```--allow-warnings```

最后执行：

```pod lib lint --allow-warnings```
 

##error: unknown: Encountered an unknown error (Unable to find a specification for 'xxxx')

在私有库中，引用了其他私有类库。因为校验podspec文件时候，默认会到CocoaPods的Specs仓库查找相关依赖类库，所以会出现找不到的错误提示。

解决方法：添加需要查找的specs仓库的地址，所以最后执行：

```pod lib lint --sources='xxxx.git, yyyy.git'```