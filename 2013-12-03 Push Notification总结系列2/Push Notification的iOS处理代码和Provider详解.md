# Push Notification的iOS处理代码和Provider详解

### 发布于2013-12-04 12:35

## Notification系列概括
1. Push Notification简介和证书说明及生成配置
2. Push Notification的iOS处理代码和Provider详解
3. Push Notification的移动客户端定位服务

这一篇文档主要描述代码实现推送通知，在最后补充一些自己在整个过程中遇到的一些问题，供以后参考，也给其他朋友一个提醒。

## 应用程序的处理代码

这里就假定已经创建了一个iOS的App，名称就暂设为MyPushNotification吧。。。

处理推送通知的代码，主要在AppDelegate.m里，故在上述假定的项目中可以找到MyPushNotificationAppDelegate.m文件。

示例代码：

```
//本地服务器地址

#define provider_server @"http://*******"

//应用启动时候注册推送通知服务，第一次安装时系统会自动提示用户

-(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    //注册推送通知功能

    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound)];

//将标记数字置为0，则不显示

    application.applicationIconBadgeNumber = 0; 

}

 

//接收从苹果服务器返回的唯一的DeviceToken，然后发送给自己的服务端

- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {

    NSString* device_token = [NSString stringWithFormat:@"%@",deviceToken];

    NSString* device_name = [[UIDevice currentDevice] name];

    NSString* device_version = [[UIDevice currentDevice] systemVersion];

    NSString* device_type = [[UIDevice currentDevice] model];

    NSString *strUrl = [NSString stringWithFormat:@"%@?action=registerDevices&device_token=%@&device_name=%@&device_version=%@&device_type=%@",

                        provider_server,device_token,device_name,device_version,device_type];

    strUrl = [strUrl stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

    NSURL *url = [NSURL URLWithString:strUrl];

     

    NSURLRequest *request = [[NSURLRequest alloc] initWithURL:url];

    //发送URL请求

    NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self];

}

 

//程序处于启动状态，或者在后台运行时，会接收到推送消息

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo

{   

    if ([[userInfo objectForKey:@"aps"] objectForKey:@"alert"]!=NULL) {

        if(application.applicationState ==UIApplicationStateActive){

 　　　　　　UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"温馨提示"  

　　　　　　　　　　　　　　message:[NSString stringWithFormat:@"\n%@",  

 　　　　　　　　　　　　　　　　　　[[userInfo objectForKey:@"aps"] objectForKey:@"alert"]]  

     　　　　　　　　　　　delegate:self  

    　　　　　　　　　　　 cancelButtonTitle:@"OK"  

　　　　　　　　　　　　　　otherButtonTitles:nil];  
 　　　　　　[alertView show];  

 　　　　　　[alertView release]; 

 

        }

    }

}
``` 

这里重点列出了几个值得注意的代理方法，当然，还可以实现更多的代理来丰富推送通知功能。

为了测试，我在实现

```
-(void)application:(UIApplication*)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
```

方法的时候，直接采用了控制台NSLog输出DeviceToken,然后将得到的字符串复制，以便在Provider中使用，因为暂时还没有搭建好本地服务器，无法采用发送url请求的方式。

只需要这几个方法就可以简单处理推送通知了。

## Provider详解

终于说到Provider了，看了很多资料，对于这一块的介绍都不是很详细，大多都是一笔带过。

确实，目前最有效的办法是直接采用开源框架来处理，我这里也不例外，但是还是想多说一下这方面的心得。

### 推荐两个开源框架

（1）Mac OS X系统下的PushMeBaby

[https://github.com/zomfg/PushMeBaby-OneMoreTime-Again](https://github.com/zomfg/PushMeBaby-OneMoreTime-Again)

这个框架的优势就是可以直接在Mac环境里使用，采用的证书为apns***.cer，详情参考上一篇的证书说明。不需要生成p12文件。使用方法简单快速，添加自己的证书文件到资源目录下，然后直接替换框架代码里面的证书变量和DeviceToken就可以了。

（2）Windows系统下的PushSharp 

[https://github.com/Redth/PushSharp](https://github.com/Redth/PushSharp)

这个框架的优势在于可适用于多终端，这意味着，可扩展性更强，不管是向什么终端设备推送，都可以实现。跟上一个框架相比，需要采用p12文件作为推送条件，注意该框架要求提供p12文件的密码，这属于上一篇证书说明中生成p12文件的内容了。

从公司的长远考虑，我采用了研究这个框架。
 

### 代码修改说明

在项目PushSharp.Sample的Program.cs中将不用的终端类型代码注释，修改“APPLE NOTIFICATIONS”下面的代码，注意修改变量appleCert的p12证书和push.RegisterAppleService中的证书密码，最后将push.QueueNotification中的DeviceToken参数改为目标字符串，就大功告成了。

先运行客户端应用程序，然后运行命令行程序后，就可以看到推送过程了。

提醒：在编译PushSharp项目时，可能会出现一堆错误提醒，别担心，很可能是由于没有开启“NuGet”。在VS中，工具——选项——包管理器——常规，选中“允许NuGet在生成期间下载缺少的程序包”。再次等待编译就可以了。

### 但是不管采用哪个框架，实现思路都是一样的

1. 搭建本地服务器，配置环境和数据库
2. 接收客户端发送来的DeviceToken，并对其进行妥善管理
3. 以证书为凭据，将目标DeviceToken和消息内容发送给APNS，请求推送通知服务
4. 可处理当客户端接收到APNS推送的消息后发来的url请求，并做相应处理。完成整个推送过程。

## 配置或者编译过程中遇到的错误

### iOS推送消息证书错误-Code=3000 "未找到应用程序的“aps-environment”的权利字符串"

这个错误应该会很普遍，涉及的疑点比较多。

### AppID含有通配符，请参考上一篇内容
### 证书生成顺序不对，请参考上一篇内容
### Target证书配置不正确
### 编译的问题，可以先将设备上的该应用删除，然后clean后重新编译项目
 

到此，Push Notification总结系列就可以结束了，但是和别人比较，总要有新内容才算进步嘛，所以在这个总结里面，加入了第三篇，定位服务的提供。而且这个功能肯定和推送服务是一起的，因为推送内容的本地化要求和按区域区分，定位服务就必不可少了。

所以，下一篇就总结一下自己研究定位服务的心得。