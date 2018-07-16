# 创建和使用私有Pods

### 2017-03-03 10:14

前一篇记录了使自己的项目支持CocoaPods管理的过程，核心的步骤就是podspec的配置和提交。

这个文件，记录了类库的详细信息，用于对类库的集成。

需要注意的一点，上一篇创建的podspec文件，被提交到了CocoaPods的Specs仓库：[https://github.com/CocoaPods/Specs.git](https://github.com/CocoaPods/Specs.git)

## 创建私有Pods的关键

创建私有Pods的关键就在于：

项目类库和Specs仓库的位置都需要是私有的，或放在内网仓库中，或放在公网私有仓库中。

## 创建步骤

类库、podspec文件的创建和仓库管理就略过了。

### 创建存放podspec文件的仓库

为了模仿CocoaPods的命名，所以可以在自己的仓库中，创建一个名为Specs的仓库。例如：[https://github.com/ALongWay/Specs.git](https://github.com/ALongWay/Specs.git)

然后将这个仓库加入本地的CocoaPods仓库，使用官方指令：

```pod repo add REPO_NAME SOURCE_URL```

如下所示，在终端执行：

```pod repo add alongway-specs https://github.com/ALongWay/Specs.git```

说明：

上面的指令中，alongway-specs是自定义的本地仓库名称，用于区别其他本地仓库。 

完成后，检查是否添加成功，如下：

```
cd ~/cocoapods/repos/alongway-specs

pod repo lint .
```

一般都是成功的。

### 将配置好的podspec文件，提交到自己的Specs仓库

将配置好的podspec文件，提交到自己的Specs仓库即可，以ALWTitleTabBar为例，如下：

```pod repo push alongway-specs ALWTitleTabBar.podspec```

等待完成push即可。

## 私有pods的使用

私有pods在podfile文件中的配置，只需要在文件顶部加入podspecs源路径：

```source 'URL_TO_REPOSITORY'```

例如：

```source https://github.com/ALongWay/Specs.git```

其他配置照旧。