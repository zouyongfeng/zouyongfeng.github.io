##导读
>现在有很多APP的tabbar上的图标在做活动时，都是在可以在服务器获取数据的，我们平常下载图片都是用的sdwebimage等第三方库，但是里面没有对于tabbaritem的下载图片的方法。不自定义tabbar的话，只能自己给tabbaritem写个从网络获取图片的功能了。

####实现方案
1.按照sdwebimage的设计思路，需要给tabbaritem写一个UITabBarItem+WebCache的分类，拓展2个下载图片的方法(选中和非选中)。

- (void)zy_setImageWithURL:(NSString *)urlString withImage:(UIImage *)placeholderImage
- (void)zy_setSelectImageWithURL:(NSString *)urlString placeholderImage:(UIImage *)placeholderImage

2.需要一个图片下载缓存器去协助UITabBarItem+WebCache去完成此功能。还是模仿sdwebimage，根据传入的url，依次从内存、本地、网络获取image，第一次从网络下载到图片时，在内存和本地都保存起来。获取到image都不往下执行了，实现如下：

- (void)zy_setImageWithURL:(NSString *)urlString withImage:(UIImage *)placeholderImage {

//1.从内存中加载图片
UIImage *image = [[ZYImageCacheManager sharedImageCacheManager] imageFromDictionary:urlString];

if (image) {
//内存有图片，就只接使用图片
self.image = [self scaleImage:[image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] toScale:0.5];
} else {
//内存没有图片，就从本地读区图片
image = [[ZYImageCacheManager sharedImageCacheManager] imageFromLocal:urlString];

if (image) {
//从本地读取到了图片，就直接使用
self.image = [self scaleImage:[image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] toScale:0.5];

//保存图片到内存中
[[ZYImageCacheManager sharedImageCacheManager] setImage:image withURL:urlString];
} else {
//从本地没有读取到图片，从网络下载
[[ZYImageCacheManager sharedImageCacheManager] imageFromNetwork:urlString complete:^(UIImage *image) {
//图片下载完成后的操作
//使用图片
self.image = [self scaleImage:[image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] toScale:0.5];
if (image == NULL) {
self.image = [placeholderImage imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
return;
}
//加载到内存
[[ZYImageCacheManager sharedImageCacheManager] setImage:image withURL:urlString];

//加载到本地
[[ZYImageCacheManager sharedImageCacheManager] saveImage:image withURL:urlString];
}];
}
}
}

我将ZYImageCacheManager设置成一个单例类，方便控制在内存中的图片，实现如下方法：

//从内存中加载图片
- (UIImage* )imageFromDictionary:(NSString* )key;

//从本地加载图片
- (UIImage* )imageFromLocal:(NSString* )urlString;

//从网络加载图片，因为是异步下载，方法结束的时候图片还没有下载完成，我们没有数据返回
- (void)imageFromNetwork:(NSString* )urlString complete:(complete)complete;

//保存图片到内存中
- (void)setImage:(UIImage* )image withURL:(NSString* )urlString;

//保存图片到本地
- (void)saveImage:(UIImage* )image withURL:(NSString* )urlString;


3.缓存到内存中，是将URLString作为key将image对象设置到字典中；缓存到本地是将URLString进行md5加密（因为url里面包含“/”，为了消除它）作为图片名字存到沙盒中；下载图片，这里使用多线程异步并发去下载：

- (void)imageFromNetwork:(NSString *)urlString complete:(complete)complete {
//异步下载图片
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
//从网络获取数据
NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:urlString]];

//从二进制生成图片
UIImage *image = [UIImage imageWithData:data];
//切回主线程
dispatch_async(dispatch_get_main_queue(), ^{
if (complete) {
complete(image);
}
});
});
}

4.同时要注意内存大小，防止内存警告。

//接受一下内存警告
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(clearImageCache) name:UIApplicationDidReceiveMemoryWarningNotification object:nil];
- (void)clearImageCache {
//清空缓存释放内存
[_imageCache removeAllObjects];
NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"Library/Caches/ImageCache"];

[[NSFileManager defaultManager] removeItemAtPath:path error:nil];
}


具体代码见我的github:https://github.com/zouyongfeng/UITabbarItem
star一下吧
