# App开发流程之自定义字体和常量使用

### 2016-09-06 13:09

## 自定义字体

之前在配置Info.plist文件时候，添加了字体的键值对，现在就可以派上用途了。

### 先将.ttf字体添加到项目中，然后配置Info.plist文件

先将.ttf字体添加到项目中，然后在Info.plist文件的Source code中，找到UIAppFonts键，增加如下代码：

```
<key>UIAppFonts</key>
    <array>
        <string>fzzy.ttf</string>
    </array>
```

当然，也可以在列表模式下，找到Fonts provided by application键，然后添加一个item，将ttf文件的名称填入其中。

### 输出字体log

在合适的地方（例如AppDelegate的应用启动后的代理方法中）增加代码如下：

```
for (NSString* familyName in [UIFont familyNames]) {
        LOG(@"familyName : %@", familyName);
        for (NSString* fontName in [UIFont fontNamesForFamilyName:familyName]) {
            LOG(@"fontName : %@", fontName);
        }
}
```

### 根据log查找字体名称

运行应用后，在控制台查看字体输出log，既然用的是fzzy字体，所以实际名称应该类似。所以找到了“FZY3JW--GB1-0”这样的字体名称

### 查看字体显示效果

需要注意的是，同一字体族中，细体、粗体、斜体等，都是不同的字体名称，可以理解为在使用不同的字体。一个ttf字体文件，一般只有一种字体，具体情况需要查看控制台输出的字体log。

## 常量使用

### define和const的比较

define宏定义可以预编译一些代码，简化逻辑，提高复用率。但是容易被覆盖，Xcode会提示。

const常量定义，是运行时分配内存和初始化，不允许修改，但是只能适用于常用数据类型的常量。

大量的宏定义肯定是会影响编译效率乃至应用体积的，配合适用应该是最佳的。

### const的用法有如下几种

1. const CGFloat a = 1.2;

2. const NSString *b = @"b";

3. NSString const *c = @"c";

4. NSString *const d = @"d";

### 说明：const右边的部分不允许再修改。

1. 第一种，a不能修改
2. 第二、三种一样，都是```*b```、```*c```不能修改，即地址
3. 第四种，是d不能修改，即内容