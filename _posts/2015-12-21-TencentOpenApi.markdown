---
layout: post
title:  "TencentOpenApi"
date:   2015-12-21 22:21:49
categories: Jekyll Update
tags: Coding
---
<!--<strong>TencentOpenAPI</strong>-->
<p>TencentOpenAPI有多坑就不说了，无奈公司要求必须非第三方SDK做分享，所以还是硬着头皮把文档看完了，总结了一下遇到的坑。</p>
<p>要做分享的话当然要先到想要分享到的app下注册appid了，做QQ分享就去腾讯开放平台注册一个，很快的，注册完以后会得到一个APP ID和APP KEY，APP KEY应该会用在第三方分享sdk上，比如SHARESDK和UmengSDK，TencentOpenApi只需要APP ID就可以了，接下来就是将最新下载的TencentOpenApi包导入项目中了，导入之后先运行一下，提示没有错误再进行下一步。</p>
这里就在你要分享的类里导入<strong>TencentOpenApi/TencentOAuth.h和TencentOpenApi/QQApiInterface.h,</strong>然后在viewDidLoad里创建一个outh，{% highlight ruby %}TencentOauth *outh = [[TencentOAuth alloc] initWithAppID:@"appid" andDelegate:delegate]{% endhighlight %},再在你要分享的按钮下加入以下代码：</br>{% highlight ruby %}
NSString *utf8String = @"www.baidu.com";
NSString *title = @"white";
NSString *description = @"you don't know";
NSString *path = [[NSBundle mainBundle] pathForResource:@"myself" ofType:@"jpg"];
NSData *data = [NSData dataWithContentsOfFile:path];
QQApiNewsObject *obj = [QQApiNewsObject objectWithURL:[NSURL URLWithString:utf8String] title:title description:description previewImageData:data];
SendMessageToQQReq *req = [SendMessageToQQReq reqWithContent:obj];
[QQApiInterface sendReq:req];{% endhighlight %}
这里的obj方法中的imageData可以使用url也可以使用MainBundle中的图片，这个看个人需求。另外分享有5种分享消息类型，多数情况是新闻形式的，即有链接，有图片，有描述，所以这里的消息形式是QQApiNewsObject。这时点击按钮会提示如下错误：

{% highlight ruby %}Undefined symbols for architecture x86_64:
"_SCNetworkReachabilityCreateWithAddress", referenced from:
+[TCOSDKReachability reachabilityWithAddress:] in TencentOpenAPI(TCOSDKReachability.o)
"_SCNetworkReachabilityCreateWithName", referenced from:
+[TCOSDKReachability reachabilityWithHostName:] in TencentOpenAPI(TCOSDKReachability.o)
"_SCNetworkReachabilityGetFlags", referenced from:
-[TCOSDKReachability currentReachabilityStatus] in TencentOpenAPI(TCOSDKReachability.o)
-[TCOSDKReachability isReachable] in TencentOpenAPI(TCOSDKReachability.o)
-[TCOSDKReachability isConnectionRequired] in TencentOpenAPI(TCOSDKReachability.o)
-[TCOSDKReachability isConnectionOnDemand] in TencentOpenAPI(TCOSDKReachability.o)
-[TCOSDKReachability isInterventionRequired] in TencentOpenAPI(TCOSDKReachability.o)
-[TCOSDKReachability isReachableViaWWAN] in TencentOpenAPI(TCOSDKReachability.o)
-[TCOSDKReachability isReachableViaWiFi] in TencentOpenAPI(TCOSDKReachability.o)
...
"_SCNetworkReachabilityScheduleWithRunLoop", referenced from:
-[TCOSDKReachability startNotifier] in TencentOpenAPI(TCOSDKReachability.o)
"_SCNetworkReachabilitySetCallback", referenced from:
-[TCOSDKReachability startNotifier] in TencentOpenAPI(TCOSDKReachability.o)
"_SCNetworkReachabilityUnscheduleFromRunLoop", referenced from:
-[TCOSDKReachability stopNotifier] in TencentOpenAPI(TCOSDKReachability.o)
"___gxx_personality_v0", referenced from:
-[TXAppidConvert InitWithAppId:] in TencentOpenAPI(AppidConvert.o)
Dwarf Exception Unwind Info (__eh_frame) in TencentOpenAPI(AppidConvert.o)
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation){% endhighlight %}这是因为导入SDK之后，还需要导入相应的依赖库，这里点击根目录，再点击其中的BuildPhases中的Link Binary With Libraries，添加SystemConfiguration.framework,libstdc++.6.0.9.tbd,目前只需要加这两个就够了，如果出现其它的类似错误还需要导入其它的依赖库，这时点击按钮，可以看到控制台输出{% highlight ruby %}-canOpenURL: failed for URL: "mqq://" - error: "This app is not allowed to query for scheme mqq"{% endhighlight %},  这是因为:近期苹果公司iOS 9系统策略更新，限制了http协议的访问，此外应用需要在“Info.plist”中将要使用的URL Schemes列为白名单，才可正常检查其他应用是否安装。受此影响，当你的应用在iOS 9中需要使用QQApi的相关能力（分享、收藏、支付、登录等）时，需要在“Info.plist”里增加如下代码：{% highlight ruby%}<key>LSApplicationQueriesSchemes</key>
<array>
<string>mqq</string>{% endhighlight %}这行代码的意思是在Info.plist文件中新加一个字典，名字为LSApplicationQueriesSchemes，然后添加数组，数组的元素就是error中的not allowed to query for scheme后面的内容，比如mqq,mqqapi,weixin等等，如果出现{% highlight ruby%}The resource could not be loaded because the App Transport Security policy requires the use of a secure connection.{% endhighlight %}这样的错误的话，则还要加一个字典，名字为App Transport Security Settings,添加一个bool元素Allow Arbitrary Loads，值设为YES。到了这一步，应该就大功告成了.

不过为了处理来自QQ的响应还应该设置一下回调，先在项目根目录里Info中的URL Types添加URL Schemes,QQ分享的形式是Tencent+APP ID，即Tencent123456789，然后在AppDelegate中添加function：{% highlight ruby %}- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation{
if ([[url absoluteString] hasPrefix:@"tencent"]){
return [QQApiInterface handleOpenURL:url delegate:self];
}
return YES;
}{% endhighlight %}加判断是为了处理不同app分享的回调。然后添加代理方法{% highlight ruby %}/**
处理来至QQ的请求
*/
- (void)onReq:(QQBaseReq *)req{

}

/**
处理来至QQ的响应
*/
//- (void)onResp:(QQBaseResp *)resp{
//    
//}

/**
处理QQ在线状态的回调
*/
- (void)isOnlineResponse:(NSDictionary *)response{

}{%endhighlight%}
至此完结。
