[TOC]

###混合工程搭建
为了项目可以支持Flutter和Native混合开发的模式，我们需要在对原生项目无侵入的条件下接入flutter，原生项目直接依赖flutter项目产物，如下图所示：
![TB1OqY3Ff1TBuNjy0FjXXajyXXa-1279-1125.png](https://upload-images.jianshu.io/upload_images/680706-09ed7848ca4d3a95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
------------------
####Flutter官方文档提供的混合方案
#####1.创建Flutter工程
安装flutter，自行百度；任意目录下执行`flutter create -t module my_flutter`，`"my_flutter" `是要创建的 Flutter 工程的名称。
#####2.通过 Cocoapods 将 Flutter 引入 现有 Native 工程
在`Podfile`添加以下下代码
```
flutter_application_path = "xxx/xxx/my_flutter"
eval(File.read(File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')), binding)
```
然后执行`pod install`
这个ruby脚本主要做下面4件事情：
-  解析 'Generated.xcconfig' 文件，获取 Flutter 工程配置信息，文件在'my_flutter/.ios/Flutter/'目录下，文件中包含了 Flutter SDK 路径、Flutter 工程路径、Flutter 工程入口、编译目录等。
- 将 Flutter SDK 中的 Flutter.framework 通过 pod 添加到 Native 工程。
- 将 Flutter 工程依赖的Native插件通过 pod 添加到 Native 工程
- 使用 post_install 这个 pod hooks 来关闭 Native 工程的 bitcode，并将 'Generated.xcconfig' 文件加入 Native 工程。
#####3.修改 Native 工程
打开Xcode工程，选择要加入 Flutter App 的 target，选择 `Build Phases`，点击顶部的 + 号，选择 `New Run Script Phase`，然后输入以下脚本
```
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" build
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" embed
```
这里执行的flutter包根目录shell脚本的作用：
- build: 根据当前 Xcode 工程的 'configuration' 和其他编译配置编译 Flutter 工程
- embed: 将 build 出来的 framework、资源包放入 Xcode 编译目录，并签名 framework

这里就有了一个问题，Flutter 工程依赖 Native工程来执行编译，影响Native工程的开发流程与打包流程，开发Native的人也需要安装Flutter环境才能调试APP
#####4.总结
以上操作可以简单的理解为，Native工程配置好脚本后，运行时会先编译Flutter项目，Flutter项目会在自己的相应目录生成Flutter.framework、依赖的Native插件等产物，最终在pod中配置好路径等参数，通过pod本地依赖的方式集成了flutter。

####实现无侵入Native Flutter 混合工程
基于官方的方案，为了实现这个目标，需要实现以下2点：

1. Flutter 工程里创建一个打包脚本，可以产生 Flutter 工程产物并上传到远程仓库；
2. 在 Native 工程用pod依赖远程仓库中的Flutter工程产物；并且保留依赖本地Flutter工程源码的功能，便于调试。

#####1.Flutter项目打包脚本
在项目目录中加入build_ios.sh文件，脚本自动打包 Flutter 工程大致分为一下几个步骤：
- `flutter_get_packages()`:检查 Flutter 环境，拉取 Flutter plugin 
- `build_flutter_app()`:编译 Flutter 工程得到产物并copy到特定文件路径下，主要逻辑和官方提供的`xcode_backend.sh`脚本差不多 
-  `flutter_copy_packages()`:得到 Flutter 产物中的 Native 插件，并copy到特定文件路径下
- `upload_product()`:release模式中将产物同步上传到git中

执行`./build_ios.h -m debug` `./build_ios.h -m release`得到不同环境的产物，并上传远程仓库

#####2.Native 依赖 Flutter 产物
这部分我们需要实现获取 Flutter 工程 release 产物，并集成到 Native 项目，并保留可以依赖本地 Flutter 工程的能力。
在原生项目中加入`flutterhelper.rb`脚本，分为如下几个步骤：
- 获取 Flutter 工程产物
- 获取 release 产物`install_release_flutter_app`:clone远程仓库中的Flutter产物到本地
- 获取 debug 产物`install_debug_flutter_app`：在 Flutter工程路径下，执行 build_ios.sh -m debug 进行打包，然后得到 debug 产物目录
- 通过 pod 引入 Flutter 工程产物`install_release_flutter_app_pod`:遍历Flutter产物目录，使用`pod sub, :path=>sub_abs_path`依赖Flutter.FrameWork、Native插件等

`podfile`中配置如下：
```
# 为true时，debug环境 为false时，release环境
FLUTTER_DEBUG_APP=true
# 如果指定了FLUTTER_APP_PATH，则此配置失效
FLUTTER_APP_URL= "http://appinstall.aiyoumi.com:8282/flutter/iOS_flutter_product.git"
# flutter git 分支，默认为master
# 如果指定了FLUTTER_APP_PATH，则此配置失效
FLUTTER_APP_BRANCH="master"
# flutter本地工程目录，绝对路径或者相对路径，如果有值则git相关的配置无效
FLUTTER_APP_PATH="/Users/zouyongfeng/ac_flutter_module"

eval(File.read(File.join(__dir__, 'flutterhelper.rb')), binding)

```

最后在jenkins中配置好打包job即可，如下：
```
cd ${WORKSPACE}
if [[ ! -d "${FLUTTER_PROJECT_Name}" ]]; then
git clone ${FLUTTER_PROJECT_GIT_REPO} ${FLUTTER_PROJECT_Name} -b ${PROJECT_GIT_BRANCH}
fi

if [[ ! -d "${FLUTTER_PRODUCT_Name}" ]]; then
git clone ${FLUTTER_PRODUCT_GIT_REPO} ${FLUTTER_PRODUCT_Name} -b ${PROJECT_GIT_BRANCH}
fi

cd ${WORKSPACE}/${FLUTTER_PRODUCT_Name}
git fetch
git reset --hard
git checkout ${PROJECT_GIT_BRANCH}
git pull --no-commit --all

cd ${WORKSPACE}/${FLUTTER_PROJECT_Name}
git fetch
git reset --hard
git checkout ${PROJECT_GIT_BRANCH}
git pull --no-commit --all
source ~/.bash_profile
sh build_ios.sh -m release
```

###与原生交互实践
####Flutter官方混合方案
#####1.Flutter调用原生
Flutter提供了FlutterMethodChannel实现了Flutter调用原生方法的功能，如下:
```
//native中
FlutterViewController* flutterViewController = [[FlutterViewController alloc] initWithProject:nil nibName:nil bundle:nil];
[flutterViewController setInitialRoute:@"myApp"];
__weak __typeof(self) weakSelf = self;
// 要与main.dart中一致
NSString *channelName = @"com.pages.your/native_get";
FlutterMethodChannel *messageChannel = [FlutterMethodChannel methodChannelWithName:channelName binaryMessenger:flutterViewController];
[messageChannel setMethodCallHandler:^(FlutterMethodCall * _Nonnull call, FlutterResult  _Nonnull result) {
if ([call.method isEqualToString:@"iOSFlutter"]) {
TargetViewController *vc = [[TargetViewController alloc] init];
[self.navigationController pushViewController:vc animated:YES];
if (result) {
result(@"返回给flutter的内容");
}
}
}];

//flutter中
// 创建一个给native的channel
static const methodChannel = const MethodChannel('com.pages.your/native_get');
_iOSPushToVC() async {
dynamic result;
result = await methodChannel.invokeMethod('iOSFlutter', '参数');
}

```
#####2.原生调用Flutter
Flutter提供了FlutterEventChannel来完成原生调用Flutter
```
// native中
FlutterEventChannel *evenChannal = [FlutterEventChannel eventChannelWithName:channelName binaryMessenger:flutterViewController];
// 代理FlutterStreamHandler
[evenChannal setStreamHandler:self];
#pragma mark - <FlutterStreamHandler>
// 这个onListen是Flutter端开始监听这个channel时的回调，第二个参数 EventSink是用来传数据的载体。
- (FlutterError* _Nullable)onListenWithArguments:(id _Nullable)arguments
eventSink:(FlutterEventSink)events {
// arguments flutter给native的参数
if (events) {
events(@"push传值给flutter的vc");
}
return nil;
}

// flutter中
// 注册一个通知
static const EventChannel eventChannel = const EventChannel('com.pages.your/native_post');
// 监听事件，同时发送参数
eventChannel.receiveBroadcastStream(12345).listen(_onEvent,onError: _onError);
String naviTitle = 'title' ;
// 回调事件
void _onEvent(Object event) {
setState(() {
naviTitle =  event.toString();
});
}
```
#####3.总结
以上就是官方提供的混合开发方案了，这个方案有一个巨大的缺点，就是在原生和Flutter页面叠加跳转时内存不断增大，因为FlutterView和FlutterViewController每次跳转都会新建一个对象，创建的Flutter页面越多内存就会暴增，尤其是在iOS上还有内存泄露的问题。
####flutter_boost混合方案
#####1.简介
![1552968436255-e781d85b-cc08-4dad-8267-a4bb94c7229c.png](https://upload-images.jianshu.io/upload_images/680706-c8e882f41dd0064d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以这样简单去理解这个方案：我们把共享的`Flutter View`当成一个画布，然后用一个`Native`的容器作为逻辑的页面。每次在打开一个容器的时候我们通过通信机制通知`Flutter View`绘制成当前的逻辑页面，然后将Flutter View放到当前容器里面。

页面栈完全由原生控制，每一个`flutter`页面对应一个原生容器（`ViewController`和`Activity`），原生端创建`FlutterRouter`实现`FLBPlatform`中的接口，flutter和原生的相互调用都会执行`FlutterRouter`中的`openPage`接口。代码如下：
```
// iOS: FlutterRouter
- (void)openPage:(NSString *)name params:(NSDictionary *)params animated:(BOOL)animated completion:(void (^)(BOOL finished))completion {
[ACRouter openWithURLString:name userInfo:params completion:^(ACRouterOutModel * _Nonnull outModel) {
[FlutterBoostPlugin.sharedInstance onResultForKey:[params objectForKey:requestIdKey] resultData:outModel.data params:@{}];
if(completion) completion(YES);
}];

}
```
flutter端建立`ACRouter`封装`flutterboost`，flutter跳转原生页面直接调用原生项目中的路由
```
// flutter中:
// 传递协议名和页面所需初始化参数
ACRouter.openUrl("mizlicai://product/normalProductDetail", {'serial': 'PI_11221'},
routeCallback: (Map<dynamic, dynamic> result) {
// 处理回调结果
print("did recieve second route result $result");
});

// Native中：
// TODO:普通产品详情
[ACRouter registerWithURLString:@"mizlicai://product/normalProductDetail" handler:^(NSDictionary * _Nullable paramsIn) {
ProductDetailViewController *vc = [[ProductDetailViewController alloc] init];
vc.serial = [paramsIn valueForKey:@"serial"];
vc.origin = [paramsIn valueForKey:@"origin"];
[[UIViewController mz_topController].navigationController pushViewController:vc animated:YES];
}];
```
flutter端和原生打开flutter页面
```
// 原生中
[ACRouter registerWithURLString:@"mizlicai://flutter/open" handler:^(NSDictionary * _Nullable paramsIn) {
NSMutableDictionary *params = [[NSMutableDictionary alloc] initWithDictionary:paramsIn[@"params"]];

FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
[vc setName:paramsIn[@"pageName"] params:params];
[[UIViewController mz_topController].navigationController pushViewController:vc animated:animated];
ACRouterCompletionBlock action = paramsIn[ACRouterParameterCompletion];
if (action) {
ACRouterOutModel *outModel = [[ACRouterOutModel alloc] init];
action(outModel);
}
}];

//flutter中
ACRouter.openUrl("mizlicai://flutter/open", {'pageName': 'userCenter','params':{},
routeCallback: (Map<dynamic, dynamic> result) {
// 处理回调结果
print("did recieve second route result $result");
});
```
#####2.协议支持
flutter可以调用原生项目组件化的路由协议([米庄iOS路由协议](http://wiki.aicaigroup.work/pages/viewpage.action?pageId=21631190))，来跳转原生页面、调用原生接口等。
#####3.网络数据请求
为了保持和原生请求框架保持同一份逻辑，使用抽象类的方式封装请求工具，Flutter启动时判断环境，使用真实请求类还是Mock请求类。
```
// main.dart
if (ApiClient.isProduction) {
ApiClient.request = RealRequest();
} else {
ApiClient.request = MockRequest();
}
```
MockRequest和RealRequest分别实现父类send方法，RealRequest通过ACRouter调用原生发起网络请求，MockRequest解析本地json
```
// 发起请求
ApiClient.request.send(Api.userCenter, HttpRequest.GET, {},
(Map response) {           
});
// RealRequest
void send(String url, String requestType, Map param, Function callback) {
param.addAll({'url': url, 'requestType': requestType});
ACRouter.openUrl(RouteCst.httpFlutterRequest, param,
routeCallback: (Map<dynamic, dynamic> result) {
callback(result);
});
}

// MockRequest
void send(String url, String requestType, Map param, Function callback) {
dynamic responseJson =
MockRequest.mock(action: getJsonName(url), param: param);
callback(responseJson);
}
```
#####4.页面导航
Flutter页面栈由原生控制，使用自己的导航栏。关闭不同页面的方法
```
// 关闭返回上一页
static Future<bool> closeCurPage()
// 返回到特定页面，使用openUrl交互
ACRouter.openUrl('mizlicai://product/closeToRoot', param,
routeCallback: (Map<dynamic, dynamic> result) {
callback(result);
});
```
#####5.原生接入
在`Podfile`中添加配置，可以切换本地，远程，debug等环境
```

platform :ios, '9.0'

# 为true时，debug环境 为false时，release环境

FLUTTER_DEBUG_APP=false

# 如果指定了FLUTTER_APP_PATH，则此配置失效

FLUTTER_APP_URL= "http://appinstall.aiyoumi.com:8282/flutter/iOS_flutter_product.git"

# flutter git 分支，默认为master

# 如果指定了FLUTTER_APP_PATH，则此配置失效

FLUTTER_APP_BRANCH="master"

# flutter本地工程目录，绝对路径或者相对路径，如果有值则git相关的配置无效

FLUTTER_APP_PATH="/Users/zouyongfeng/ac_flutter_module"

eval(File.read(File.join(__dir__, 'flutterhelper.rb')), binding)

```
AppDelegate中，初始化`flutterboost`，传入`FlutterRouter`
```
#import "FlutterRouter.h"
- (void)startFlutter {

[FlutterBoostPlugin.sharedInstance startFlutterWithPlatform:[FlutterRouter sharedRouter]

onStart:^(FlutterViewController *fvc) {
}];

}
```
