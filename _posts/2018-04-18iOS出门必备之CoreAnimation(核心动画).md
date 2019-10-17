
- 前段时间接触到了一个牛逼的动画框架[POP](https://github.com/facebook/pop),本来想来装装逼,突然发现,苹果大大的CoreAnimation我还不会用呢!
- 依稀记得乔帮主在2007年的WWDC大会上亲自为你演示Core Animation的强大：[点击查看视频](http://v.youku.com/v_show/id_XMzQ2MTcwNDQ0.html)(不好意思,又装逼了)
- 言归正传,我只是来温习一下CoreAnimation,还望路过的大神不要吐槽我太low
- [GitHub项目地址](https://github.com/coderQuanjun/POPAnimationDemo)

<!-- more -->


## Core Animation简介

* `Core Animation`，中文翻译为核心动画，它是一组非常强大的动画处理API，使用它能做出非常炫丽的动画效果，而且往往是事半功倍。也就是说，使用少量的代码就可以实现非常强大的功能。
* `Core Animation`可以用在`Mac OS X和iOS`平台。
* `Core Animation`的动画执行过程都是在后台操作的，不会阻塞主线程。
* 要注意的是，Core Animation是直接作用在`CALayer`上的，并非UIView
* 通过调用`CALayer`的`addAnimation:forKey:`方法增加`CAAnimation`对象到`CALayer`中，这样就能开始执行动画了
* 通过调用`CALayer`的`removeAnimationForKey:`方法可以停止`CALayer`中的动画

### Core Animation及其相关属性
- 要想执行动画，就必须初始化一个`CAAnimation`对象。
- 一般情况下，我们使用的比较多的是`CAAnimation`的子类，因此，先大致看看`CAAnimation`的继承结构
- 黑线代表继承，黑色文字代表类名，白色文字代表属性。其中`CAMediaTiming`是一个协议(`protocol`)


![Core Animation结构划分.png](http://upload-images.jianshu.io/upload_images/4122543-7bc6a9dcfc3ff80f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

<div class="note warning"><p>需要注意</p></div>

- CAAnimation是所有动画类的父类，但是它不能直接使用，应该使用它的子类
- CAPropertyAnimation也是不能直接使用的，也要使用它的子类
- 能用的动画类只剩下4个：CABasicAnimation、CAKeyframeAnimation、CATransition、CAAnimationGroup

### 常用属性

**1). `removedOnCompletion`：默认为true，代表动画执行完毕后就从图层上移除**
- 图形会恢复到动画执行前的状态。如果想让图层保持显示动画执行后的状态，那就设置为false，不过还要设置`fillMode`为`kCAFillModeForwards`

**2). timingFunction：控制动画运行的节奏**

```objc
/** timingFunction可选的值 **/
@available(iOS 2.0, *)
public let kCAMediaTimingFunctionLinear: String
//1.(匀速): 在整个动画时间内动画都是以一个相同的速度来改变

@available(iOS 2.0, *)
public let kCAMediaTimingFunctionEaseIn: String
//2. (渐进): 缓慢进入, 加速离开

@available(iOS 2.0, *)
public let kCAMediaTimingFunctionEaseOut: String
//3. (渐出): 快速进入, 减速离开

@available(iOS 2.0, *)
public let kCAMediaTimingFunctionEaseInEaseOut: String
//4. (渐进渐出): 缓慢进入, 中间加速, 减速离开

@available(iOS 3.0, *)
public let kCAMediaTimingFunctionDefault: String
//5. (默认): 效果基本等同于EaseOut(渐出)

```
**3). fillMode决定当前对象在非active时间段的行为。**
- 要想fillMode有效，需设置removedOnCompletion = false
- fillMode可选的值

```objc
/* `fillMode' options. */
@available(iOS 2.0, *)
public let kCAFillModeForwards: String
//1. 当动画结束后，layer会一直保持着动画最后的状态

@available(iOS 2.0, *)
public let kCAFillModeBackwards: String
//2. 设置为该值，将会立即执行动画的第一帧，不论是否设置了 beginTime属性。观察发现，设置该值，刚开始视图不见，还不知道应用在哪里

@available(iOS 2.0, *)
public let kCAFillModeBoth: String
//3. 该值是 kCAFillModeForwards 和 kCAFillModeBackwards的组合状态; 动画加入后开始之前，layer便处于动画初始状态，动画结束后layer保持动画最后的状态

@available(iOS 2.0, *)
public let kCAFillModeRemoved: String
//4. 默认值，动画将在设置的 beginTime 开始执行（如没有设置beginTime属性，则动画立即执行），动画执行完成后会将layer的改变恢复原状
```
**4). `delegate`：动画代理，用来监听动画的执行过程**

```objc
public protocol CAAnimationDelegate : NSObjectProtocol {

    // 动画开始执行的时候触发这个方法
    @available(iOS 2.0, *)
    optional public func animationDidStart(_ anim: CAAnimation)

    // 动画执行完毕的时候触发这个方法
    @available(iOS 2.0, *)
    optional public func animationDidStop(_ anim: CAAnimation, finished flag: Bool)
}
```
**5). 其他相关属性**

```objc
duration    动画的时长
repeatCount    重复的次数。不停重复设置为 HUGE_VALF
repeatDuration    设置动画的时间。在该时间内动画一直执行，不计次数。
beginTime    指定动画开始的时间。从开始延迟几秒的话，设置为【CACurrentMediaTime() + 秒数】 的方式
timingFunction    设置动画的速度变化
autoreverses    动画结束时是否执行逆动画
fromValue    所改变属性的起始值(Swift中为Any类型,OC中要包装成NSValue对象)
toValue    所改变属性的结束时的值(类型与fromValue相同)
byValue    所改变属性相同起始值的改变量(类型与fromValue相同)
```

## CABasicAnimation
- CABasicAnimation是CAPropertyAnimation的子类，使用它可以实现一些基本的动画效果，它可以让CALayer的某个属性从某个值渐变到另一个值。下面就用CABasicAnimation实现几个简单的动画

### 平移动画
#### 方法一: 改变label的position

```objc
let caBasic = CABasicAnimation(keyPath: "position")
caBasic.duration = 2
caBasic.fromValue = redLabel.layer.position
caBasic.toValue = CGPoint(x: kScreenWidth - 50, y: 200)
caBasic.delegate = self
caBasic.isRemovedOnCompletion = false
caBasic.fillMode = kCAFillModeForwards
redLabel.layer.add(caBasic, forKey: "redLabel1")
```
* 初始化方法中是@"position"，说明要修改的是CALayer的position属性，也就是会执行平移动画
* 默认情况下，动画执行完毕后，动画会自动从CALayer上移除，CALayer又会回到原来的状态。为了保持动画执行后的状态，可以加入第6、7行代码
* 第8行后面的@"redLabel1"是给动画对象起个名称，以后可以调用CALayer的removeAnimationForKey:方法根据动画名称停止相应的动画
* 遵循的代理方法

```objc
extension ViewController: CAAnimationDelegate {
    //开始执行
    func animationDidStart(_ anim: CAAnimation) {
        print("开始动画--layer:", redLabel.layer.position)
    }
    //结束之行
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        print("结束动画--layer:", redLabel.layer.position)
    }
}


//打印结果为:
//开始动画--layer: (35.0, 213.0)
//结束动画--layer: (35.0, 213.0)

```
> 从打印信息可以看出，实际上，动画执行完毕后，并没有真正改变CALayer的position属性的值！

#### 方法二. 

```objc
 let basic = CABasicAnimation(keyPath: "transform")
basic.duration = 2
let form = CATransform3DMakeTranslation(350, 400, 0)
basic.toValue = form
blueLabel.layer.add(basic, forKey: "blueLabel")
```

### 旋转动画

```objc
let basic1 = CABasicAnimation(keyPath: "transform")
basic1.duration = 1
basic1.toValue = CATransform3DMakeRotation(0.25, 0, 0, 1)
basic1.isRemovedOnCompletion = false
basic1.fillMode = kCAFillModeForwards
blueLabel.layer.add(basic1, forKey: "basic1")
```
- 可以不用设置fromValue，这里只设置了toValue

### 缩放动画
- CALayer的宽度从0.5倍变为2倍
- CALayer的高度从0.5倍变为1.5倍

```objc
let basic1 = CABasicAnimation(keyPath: "transform")
basic1.duration = 1
basic1.toValue = CATransform3DMakeScale(0.5, 0.5, 1)
basic1.toValue = CATransform3DMakeScale(2, 1.5, 1)
basic1.isRemovedOnCompletion = false
basic1.fillMode = kCAFillModeForwards
blueLabel.layer.add(basic1, forKey: "basic1")
```
> 
- CABasicAnimation虽然能够做很多基本的动画效果，但是有个局限性，只能让CALayer的属性从某个值渐变到另一个值，仅仅是在2个值之间渐变
- 总结一些常用的animationKeyPath值的

值 | 说明 | 使用形式
---|---|---
transform.scale | 比例转化 | 0.5
transform.scale.x | 宽的比例 | 0.5
transform.rotation.x | 围绕x轴旋转 | @(M_PI_4)(OC), 0.25(Swift)
cornerRadius  |	圆角的设置 | 30
backgroundColor | 背景颜色的变化 | 	UIColor.purpleColor.cgColor
bounds | 大小，中心不变 | CGRect
position | 位置(中心点的改变) | CGPoint
contents | 内容，比如UIImageView的图片 | imageAnima.toValue = UIImage(named: "toImage")?.cgImage
opacity | 透明度 | 0.7
contentsRect.size.width | 横向拉伸缩放 | 最好是0~1之间的

## CAKeyframeAnimation——关键帧动画
- 关键帧动画，也是`CAPropertyAnimation`的子类，与`CABasicAnimation`的区别是：
  - `CABasicAnimation`只能从一个数值（fromValue）变到另一个数值（toValue）
  - 而`CAKeyframeAnimation`会使用一个Array保存这些数值
- 属性说明：
  - `values`：上述的Array对象。里面的元素称为“关键帧”(keyframe)。动画对象会在指定的时间（duration）内，依次显示values数组中的每一个关键帧
  - `path`：可以设置一个`CGPathRef、CGMutablePathRef`，让图层按照路径轨迹移动。path只对CALayer的`anchorPoint`和`position`起作用。如果设置了path，那么values将被忽略
  - `keyTimes`：可以为对应的关键帧指定对应的时间点，其取值范围为0到1.0，keyTimes中的每一个时间值都对应values中的每一帧。如果没有设置keyTimes，各个关键帧的时间是平分的
  - `calculationMode`: 该属性决定了物体在每个子路径下是跳着走还是匀速走，跟`timeFunctions`属性有点类似
    - `kCAAnimationLinear`默认值,表示当关键帧为座标点的时候,关键帧之间直接直线相连进行插值计算;
    - `kCAAnimationDiscrete` 离散的,就是不进行插值计算,所有关键帧直接逐个进行显示;
    - `kCAAnimationPaced` 使得动画均匀进行,而不是按`keyTimes`设置的或者按关键帧平分时间,此时`keyTimes`和timingFunctions`无效;
    - `kCAAnimationCubic` 对关键帧为座标点的关键帧进行圆滑曲线相连后插值计算,对于曲线的形状还可以通过`tensionValues,continuityValues,biasValues`来进行调整自定义主要目的是使得运行的轨迹变得圆滑;
    - `kCAAnimationCubicPaced` 看这个名字就知道和`kCAAnimationCubic`有一定联系,其实就是在`kCAAnimationCubic`的基础上使得动画运行变得均匀,就是系统时间内运动的距离相同,此时`keyTimes`以及`timingFunctions`也是无效的.
- `CABasicAnimation`可看做是只有2个关键帧的`CAKeyframeAnimation`

### values方式

```objc
let key = CAKeyframeAnimation(keyPath: "position")
key.duration = 3
key.repeatCount = HUGE //无线循环
key.calculationMode = kCAAnimationPaced
key.values = [redLabel.frame.origin, CGPoint(x: 180, y: 70), CGPoint(x: 180, y: 200), redLabel.frame.origin]
key.keyTimes = [NSNumber(value: 0.0), NSNumber(value: 0.6), NSNumber(value: 0.7), NSNumber(value: 0.8)]
redLabel.layer.add(key, forKey: "key")
```

## CASpringAnimation
- `CASpringAnimation`是iOS 9 新出的
- `CASpringAnimation` 继承于`CABaseAnimation`
- `CASpringAnimation`是苹果专门解决开发者关于弹簧动画的这个需求而封装的类。

### CASpringAnimation相关属性

```objc
//1. 质量，影响图层运动时的弹簧惯性，质量越大，弹簧拉伸和压缩的幅度越大, 默认值: 1
open var mass: CGFloat

//2. 刚度系数(劲度系数/弹性系数)，刚度系数越大，形变产生的力就越大，运动越快(默认值: 100)
open var stiffness: CGFloat

//3. 阻尼系数，阻止弹簧伸缩的系数，阻尼系数越大，停止越快(默认值: 10)
open var damping: CGFloat

//4. 初始速率，动画视图的初始速度大小, 默认0
//速率为正数时，速度方向与运动方向一致，速率为负数时，速度方向与运动方向相反(默认值: 0)
open var initialVelocity: CGFloat

//5. 估算时间 返回弹簧动画到停止时的估算时间，根据当前的动画参数估算(只读)
open var settlingDuration: CFTimeInterval { get }

```

### 示例代码

```objc
let spring = CASpringAnimation(keyPath: "position.y")
spring.mass = 5
spring.stiffness = 100
spring.damping = 5
spring.initialVelocity = 2
spring.fromValue = blueLabel.layer.position.y
spring.toValue = kScreenHeight - 150
spring.duration = spring.settlingDuration
blueLabel.layer.add(spring, forKey: "spring")
```

## CAAnimationGroup动画组
- 是CAAnimation的子类，可以保存一组动画对象，将CAAnimationGroup对象加入层后，组中所有动画对象可以同时并发运行
- 属性说明：
  - animations：用来保存一组动画对象的Array
- 默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的beginTime属性来更改动画的开始时间

### 代码示例: 
- 同时执行：平移、缩放、位移动画 -> 使用动画组

```objc
    //动画组
    fileprivate func getCAAnimationGroup(){
        //0. 初始化动画组
        let group = CAAnimationGroup()
        
        //1. 平移动画
        let basic1 = CABasicAnimation(keyPath: "position")
        basic1.fromValue = blueLabel.layer.position
        basic1.toValue = CGPoint(x: CGFloat(arc4random_uniform(200)), y: CGFloat(arc4random_uniform(500)))
        
        //2. 缩放动画
        let basic2 = CABasicAnimation(keyPath: "transform.scale")
        var scale: CGFloat = 0.1
        scale = scale < 1 ? 1.5 : 0.5
        basic2.toValue = scale
        
        //3. 旋转动画
        let basic3 = CABasicAnimation(keyPath: "transform.rotation")
        basic3.toValue = CGFloat(arc4random_uniform(360)) / 180.0
        
        //4. 添加到动画组
        group.animations = [basic1, basic2, basic3]
        //取消反弹
        group.isRemovedOnCompletion = false
        group.fillMode = kCAFillModeForwards
        group.duration = 0.5
        blueLabel.layer.add(group, forKey: "group")
    }

```

## 转场动画——CATransition
- `CATransition`是`CAAnimation`的子类，用于做转场动画，能够为layer层提供移出屏幕和移入屏幕的动画效果。
- iOS比Mac OS X的转场动画效果少一点
`UINavigationController`就是通过`CATransition`实现了将控制器的视图推入屏幕的动画效果
- 动画属性:
  - `type`：动画过渡类型
  - `subtype`：动画过渡方向
  - `startProgress`：动画起点(在整体动画的百分比)
  - `endProgress`：动画终点(在整体动画的百分比)

### `type`和`subtype`属性说明

```objc
/* type类型 */
@available(iOS 2.0, *)
public let kCATransitionFade: String
//交叉淡化过渡

@available(iOS 2.0, *)
public let kCATransitionMoveIn: String
//新视图移到旧视图上面

@available(iOS 2.0, *)
public let kCATransitionPush: String
//新视图把旧视图推出去

@available(iOS 2.0, *)
public let kCATransitionReveal: String
//将旧视图移开,显示下面的新视图


/* subtypes类型 */
@available(iOS 2.0, *)
public let kCATransitionFromRight: String
//从右侧转场

@available(iOS 2.0, *)
public let kCATransitionFromLeft: String
//从左侧转场

@available(iOS 2.0, *)
public let kCATransitionFromTop: String
//从上部转场

@available(iOS 2.0, *)
public let kCATransitionFromBottom: String
//从底部转场
```

<div class="note warning"><p> 注意：</p></div>

- 除了上述四种效果之外,还有很多私有API效果，使用的时候要小心，可能会导致app审核不被通过
- 使用的时候要以字符串的形式

```objc
cube     //立方体翻滚效果
oglFlip  //上下左右翻转效果
suckEffect   //收缩效果，如一块布被抽走(不支持过渡方向)
rippleEffect //滴水效果(不支持过渡方向)
pageCurl     //向上翻页效果
pageUnCurl   //向下翻页效果
cameraIrisHollowOpen  //相机镜头打开效果(不支持过渡方向)
cameraIrisHollowClose //相机镜头关上效果(不支持过渡方向)

```

> 效果参考


![各参数动画效果.png](http://upload-images.jianshu.io/upload_images/4122543-48e5ec518c8f535e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

### 代码示例:
- 展示立方体翻滚效果的图片浏览

#### 初始化变量

```objc
//初始化变量
fileprivate var imageView = UIImageView(frame: UIScreen.main.bounds)
fileprivate var currentIndex = 0

```

#### 需要在`viewDidLoad`中调用一下方法

```objc
//转场动画
    fileprivate func imageCATransition(){
        //0.初始化ImageView
        imageView.isUserInteractionEnabled = true
        imageView.image = UIImage(named: "0.jpg")
        view.addSubview(imageView)
        
        //1. 添加滑动手势
        let left = UISwipeGestureRecognizer(target: self, action: #selector(leftSwipe(gesture:)))
        left.direction = .left
        imageView.addGestureRecognizer(left)
        let right = UISwipeGestureRecognizer(target: self, action: #selector(rightSwipe(gesture:)))
        right.direction = .right
        imageView.addGestureRecognizer(right)
    }

```

#### 滑动后执行的方法

```objc
    //MARK: 手势相关方法
    //左滑
    @objc fileprivate func leftSwipe(gesture: UIGestureRecognizer) {
        print("左滑动")
        transitionAnimation(isNext: true)
    }
    //右滑
    @objc fileprivate func rightSwipe(gesture: UIGestureRecognizer) {
        print("右滑动")
        transitionAnimation(isNext: false)
    }
    
    //设置转场动画
    fileprivate func transitionAnimation(isNext: Bool){
        let transition = CATransition()
        transition.type = kCATransitionFade
        transition.subtype = isNext ? kCATransitionFromRight : kCATransitionFromLeft
        transition.duration = 1
        imageView.image = getImage(isNext)
        imageView.layer.add(transition, forKey: "transition")
    }
    
    //获取下/上一张图片
    fileprivate func getImage(_ isNext: Bool) -> UIImage {
        currentIndex = isNext ? currentIndex + 1 : currentIndex - 1
        currentIndex = currentIndex < 0 ? 7 : currentIndex
        currentIndex = currentIndex > 7 ? 0 : currentIndex
        return UIImage(named: "\(currentIndex)" + ".jpg")!
    }

```

## 总结
- 核心动画给我们展示的只是一个假象，layer的的frame、bounds、position并不会在动画完毕之后发生改变。
- UIView封装的动画，会使会真实修改view的一些属性
- 以上就是小编总结的关于Core Animation核心动画的相关分类
- 总结的知识点比较简单, 个人感觉有点low
- 如有不足之处,还望路过的大神多多指教
