

- 在开发过程中一定会用到一些第三方框架, 只要安装了`CocoaPods`, 然后通过`pod install`命令, 就可以集成框架到项目中了
- 可是如果想要把自己的框架或者组件也开源出去, 让别人也可以使用, 那该如何入手 ?
- 对于`CocoaPods`还不是很了解的或者没有安装的童鞋, 可自行百度或者参考[用CocoaPods做程序的依赖](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)

<!-- more -->

## 搭建框架
### 创建仓库
- `CocoaPods`项目的源码在`Github`上管理,所以第一步我们需要创建一个属于自己的仓库
- 根据图下图所示创建自己的项目

![创建仓库](https://upload-images.jianshu.io/upload_images/4122543-d600fd516421d3a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/714/format/webp)

### 上传文件
- 要开发框架必然就要上传文件, 这里推荐`SourceTree`和`GitHub`客户端, 当然也可以使用终端命令上传
- 使用`git`管理工具我们这里暂不赘述, 不懂得可以自行百度
- 终端使用`git`命令上传, 主要命令如下

```objc
//cd到当前文件夹
// 创建本地仓库
git init
// 添加名称为origin的远程连接
git remote add origin '你的github项目地址'
// 将本地代码加入本地仓库里
git add .
// 提交修改到本地仓库
git commit -m '你的修改记录'
// 推送master分支的代码到名称为origin的远程仓库
git push origin master
//本地打标签备份
git tag "v0.0.1"
//提交标签
git push --tags
//删除本地标签
git tag -d 标签名称
//删除远程标签
git push origin: 标签名称
```


### 使用CocoaPods
#### 检索第三方框架

- 使用命令`pod search xxx`
- 从本地缓存的--第三方框架描述信息--生成的检索文件中检索到响应的框架信息
- 本地缓存的框架描述信息位置: `~/Library/Caches/CocoaPods/search_index.json`
- 如执行命令时出现错误, 可删除改索引文件, 执行命令: `rm ~/Library/Caches/CocoaPods/search_index.json`


#### 安装第三方框架
- 创建`Podfile`文件, 到自己工程内(一级目录): `pod init`
  - `Podfile`文件: 答: 其实就是使用`ruby`语法编写的 "框架依赖描述文件"; 就是告诉`cocoapods`需要下载集成哪些框架
- 安装框架
  - `pod install`
  - `cocoapods`在1.0.1以后版本, 会直接就是根据`Podfile`文件找到, 框架信息, 然后下载集成
  - 但是1.0.1之前的版本, 会先更新本地的框架描述信息(非常耗时), 然后再根据文件下载
  - 下载完成后会生成一个`Podfile.lock`文件, 记录着上一次下载的框架最新版本
- `pod install`和`pod update`
  - `pod install`
    - 如果`Podfile.lock`文件存在, 直接从此文件中读取框架信息下载安装
    - 如果不存在, 依然会读取`Podfile`文件内的框架信息, 下载好之后, 再根据下载好的框架信息, 生成`Podfile.lock`文件
  - `pod update`
    - 不管`Podfile.lock`是否存在, 都会读取`Podfile`文件的的框架信息去下载
    - 下载好之后, 再根据下载好的框架信息, 生成`Podfile.lock`文件
  - 主要区别在于, `Podfile`文件内的框架信息, 版本描述没有指定具体版本
  - 一般情况下, 每个人从共享库把项目下载下来之后, 都会执行`pod install`命令安装!这样可以保证大家使用的第三方框架版本一致!!如果以后大家需要统一升级第三方框架, 那么每个人在执行 pod update


#### CocoaPods相关操作

```
//创建Podfile文件
pod init
//搜索框架
pod search
//安装框架
pod install
//更新框架
pod update

//初始化(下载服务器中所有的第三方框架信息缓存到电脑本地)
pod setup
//查看第三方框架仓库源
pod repo
//移除仓库源
pod repo remove master
//添加仓库源
//pod repo add master https://xxxx....
```

#### `CocoaPods`相关路径

```
//索引缓存路径
//如果发现框架信息本地已经缓存, 但是就是无法搜索框架, 可以删除这个索引文件, 重新生成
 ~/Library/Caches/CocoaPods/
 
 //pod命令安装路径
 /usr/local/bin
 
 //pod框架索引信息缓存路径
 /Users/apple/.cocoapods/repos/master
```



### 创建Podspec描述文件
- 该文件为`Cocoapods`依赖库的描述文件，每个`Cocoapods`依赖库必须有且仅有那么一个描述文件
- 简单地讲就是让`CocoaPods`搜索引擎知道你的代码的作者、版本号、源代码地址、依赖库等信息的文件
- 文件名称要和我们想创建的依赖库名称保持一致

```objc
pod spec create 框架名字

// 示例:
pod spec create TitanModel
```

- 该命令将在本目录产生一个名为`TitanModel.podspec`文件
- 可用`Sublime Text`或者`Atom`打开该文件，里面已经有非常丰富的说明文档, 但是很多都是我们不需要的
- 官方`Podspec`文件的编写格式可参考 [Podspec Syntax Reference](https://guides.cocoapods.org/syntax/podspec.html)
- 下面介绍如何声明第三方库的代码目录和资源目录，还有该第三方库所依赖`ios`核心框架和第三方库
- 去掉文件中的一些注释信息, 可以看到也就剩下以下内容了

```ruby
Pod::Spec.new do |s|
    s.name          = 'TitanModel' #项目名
    s.version       = '0.1.0' #相应的版本号
    s.summary       = 'A short description of YJDemoSDK.' #简述
    s.description   = <<‐ DESC #详细描述
    TODO: Add long description of the pod here.
                      DESC
    s.homepage      = 'https://github.com/CoderTitan/TitanModel' #项目主页
    s.license       = { :type => 'MIT', :file => 'LICENSE' } #开源协议
    s.author        = { 'CoderTitan' => 'quanjunt@163.com' } #作者
    s.platform      = :ios, '8.0' #支持的平台
    s.requires_arc  = true #arc和mrc选项
    s.libraries     = 'z', 'sqlite3' #表示依赖的系统类库，比如libz.dylib等
    s.frameworks    = 'UIKit','AVFoundation' #表示依赖系统的框架
    s.ios.vendored_frameworks = 'TKBase/TKBase.framework' # 依赖的第三方/自己的framework
    s.vendored_libraries = 'Library/Classes/libWeChatSDK.a' #表示依赖第三方/自己的静态库（比如libWeChatSDK.a）
    #依赖的第三方的或者自己的静态库文件必须以lib为前缀进行命名，否则会出现找不到的情况，这一点非常重要

    #平台信息
    s.platform      = :ios, '7.0'
    s.ios.deployment_target = '7.0'

    #文件配置项
    s.source        = { :git => 'https://github.com/CoderTitan/TitanModel.git', :tag => s.version.to_s }
    #配置项目的目标路径，如果不是本地开发，pod init/update会从这个路去拉去代码

    s.source_files = 'TitanModel/Classes/**/*' #你的源码位置
    s.resources     = ['TitanModel/Assets/*'] #资源，比如图片，音频文件等
    s.public_header_files = 'TitanModel/Classes/TitanModel.h'   #需要对外开放的头文件

    #依赖的项目内容 可以多个
    s.dependency 'MJExtension'
    s.dependency 'AFNetworking'

end
```

- `s.name`：名称，`pod search`搜索的关键词,注意这里一定要和`.podspec`的名称一样,否则报错
- `s.version`：版本号，`to_s`：返回一个字符串
- `s.summary`: 项目简短的简介
- `s.description`: 这个是详细的描述, 要注意的是字数要比`summary`的长, 否则上传的时候可能会爆出警告
- `s.homepage`: 项目主页地址
- `s.license`: 许可证
- `s.author`: 作者
- `s.source`: 项目源码所在地址
- `s.platform`: 项目支持平台
- `s.requires_arc`: 是否支持`ARC`
- `s.source_files`: 需要包含的源文件
- `s.public_header_files`: 需要包含的头文件
- `s.ios.deployment_target`: 支持的`pod`最低版本
- `s.social_media_url`: 社交网址
- `s.resources`: 资源文件
- `s.dependency`: 依赖库，不能依赖未发布的库

> `source_files`写法及含义

```objc
"TitanModel"
"Classes/**/*.{h,m}"
```

- `*`表示匹配所有文件
- `*.{h,m}`表示匹配所有以`.h`和`.m`结尾的文件
- `**`表示匹配所有子目录


### 将自己的项目打成`tag`

- 因为`cocoapods`是依赖`tag`版本的,所以必须打`tag`,以后再次更新只需要把你的项目打一个`tag`，然后修改`.podspec`文件中的版本接着提交到`cocoapods`官方就可以了
- 要注意的是, 这里提交的版本号要和`TitanModel.podspec`文件中的版本号一致

```
git tag "v0.0.1"  
git push --tags
```

### 上传`Podspec`
- `Podspec`修改完成后, 上传到服务器时, 我们需要使用`trunk`进行上传
- 首先要注册`trunk`, 在注册`trunk`之前，我们需要确认当前的`CocoaPods`版本是否足够新。`trunk`需要`pod`在`0.33`及以上版本，如果你不满足要求, 需要重新安装`pod`
- 更新结束后，我们开始注册`trunk`, 可参考官方文档[Getting setup with Trunk](https://guides.cocoapods.org/making/getting-setup-with-trunk.html)
- 终端输入以下命令

```
pod trunk register 邮箱地址 '用户名' --description='描述'

// 示例
pod trunk register quanjunt@163.com 'CoderTitan' --description='macbook'
```

执行该命令后, 你的邮箱会受到一封邮件, 但是邮件要到垃圾邮件中才能找到, 打开邮件找到邮件中的网址并打开

![image](https://upload-images.jianshu.io/upload_images/4122543-a7ec0659f716c755.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

如果打开邮件中的链接和下面的页面一样, 则表示注册成功

![注册成功](https://upload-images.jianshu.io/upload_images/4122543-cc70f97600d6b4bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/785/format/webp)

最后输入如下命令

```
pod trunk push TitanModel.podspec
```

时间较长，耐性等待，大概5-10分钟, 成功后结果如下

![trunk](https://upload-images.jianshu.io/upload_images/4122543-de1a8d5fe072d85b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

- 上面图片中可以看到执行了`Updating spec repo master`命令, 该命令主要就是更新本地的`Specs`文件
- 查看文件夹位置, 打开访达文件夹, `Shift+command+G`快捷键, 打开前往文件夹操作, 输入如下目录即可查看

```
~/.cocoapods/repos/master/Specs
```

### 测试自己的cocoapods

- 终端输入`pod search TitanModel`查看
- 但是如果输入上述命令后, 终端输出如下错误

```
[!] Unable to find a pod with name, author, summary, or description matching `TitanModel`
```

这是因为你的框架已经上传, 但是你的本地的搜索文件`search_index.json`没有更新, 所以搜索不到, 可以执行下面命令删除`search_index.json`文件

```
rm ~/Library/Caches/CocoaPods/search_index.json
```

- 也可以直接找到该文件删除
- 查看文件夹位置, 打开访达文件夹, `Shift+command+G`快捷键, 打开前往文件夹操作, 输入如下目录即可查看

```
~/Library/Caches/CocoaPods/
```

<div class="note success"><p>搜索成功</p></div>

![search](https://upload-images.jianshu.io/upload_images/4122543-860729c04860ca64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/649/format/webp)


## 错误整理
### 版本号
- 设置版本号的时候一般有两种方式, 一种是前面带`v`的, 如: `v0.0.1`;另 一种是前面不带`v`的, 如:` 0.0.1`
- 因为`v`而导致的报错

```
warning: Could not find remote branch 0.0.1 to clone.

fatal: Remote branch 0.0.1 not found in upstream origin
```

为解决以上问题, 设置版本号的方式和`spec`文件内的版本号方式一定要一致
#### 不带v方式:
设置版本号时:

```
// 这里设置时, 不要带v
git tag '0.0.1'  
git push --tags  
```

`spec`文件中
```
//这里不要带v
s.version = "0.0.2"
s.source = { :git => "https://github.com/CoderTitan/TitanModell.git", :tag => s.version }
//这里的tag也可以设置成具体的版本号, 只要与上面一样就好
```

#### 带v方式:
设置版本号时:

```
// 这里设置时, 要带v
git tag 'v0.0.1'  
git push --tags  
```

`spec`文件中

```
//这里要带v
s.version = "v0.0.2"
s.source = { :git => "https://github.com/CoderTitan/TitanModell.git", :tag => "v#{s.version} }
//这里的tag也可以设置成具体的版本号, 只要与上面一样就好
```

## 总结

最后对上述涉及到的终端命令做一个简单的总结

### 终端命令
1. 开源库发布之后，需要给项目打上`tag`

```
git tag "0.0.1"  
git push --tags
```

2. 进入到项目根目录下，创建`podspec`文件

```
pod spec create TitanModel
```

3. 编辑`podspec`文件中的相关信息，有两个比较重要的地方`s.source`和`s.source_files`, 修改完成后, 验证是否有误

```
pod spec lint TitanModel.podspec
```

4. 注册`pod trunk`

```
pod trunk register orta@cocoapods.org 'Orta Therox' --description='macbook air'
```

5. 发布到`trunk`

```
pod trunk push TitanModel.podspec
```

6. 搜索发布的框架

```
pod search TitanModel
```
