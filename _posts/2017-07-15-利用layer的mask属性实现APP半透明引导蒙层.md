###如今APP中新版本都会有些半透明蒙层引导，例如下面这种的。
![屏幕快照 2017-07-07 上午11.32.51.png](http://upload-images.jianshu.io/upload_images/680706-1fa3290e94d34533.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###之前我都是用一张图片覆盖到相应的位置来完成需求，还要考虑不同屏幕的图片拉伸，而且图片上的内容固定，很不灵活。后来采用layer的mask属性来让需要显示的地方透明，完美的实现这些需求。

- mask属性按照我的理解就是显示maskLayer与父layer非透明部分的交集，所以中间的没有path的透明部分就不显示了。

- 代码实现，self就是占整个屏幕的黑色蒙层，这里的bezierPathByReversingPath可以生成反向的path，frame是需要透明显示的位置


UIBezierPath *path = [UIBezierPath bezierPathWithRect:kScreen_BOUNDS];
[path appendPath: [[UIBezierPath bezierPathWithRoundedRect:frame cornerRadius:4] bezierPathByReversingPath]];
CAShapeLayer *shapeLayer = [CAShapeLayer layer];
shapeLayer.path = path.CGPath;
self.layer.mask = shapeLayer;
