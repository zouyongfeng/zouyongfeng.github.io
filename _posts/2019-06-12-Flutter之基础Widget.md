

![Flutter](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/widget_base.png?x-oss-process=style/titanjun)



<!--more-->


- [Flutter和Dart系列文章](https://www.titanjun.top/categories/Flutter%E7%AC%94%E8%AE%B0/)
- [项目GitHub地址](https://github.com/CoderTitan/Flutter_Widget)
- `Flutter`作为一种全新的响应式，跨平台，高性能, 完全免费、开源的移动开发框架
- `Widget`是`Flutter`开发中的主要组成部分, 是`Flutter`的基础, `Flutter`的核心设计思想便是: 一切皆`Widget`
- `Flutter`中的`widget`的概念更广泛，它不仅可以表示`UI`元素，也可以表示一些功能性的组件如：用于手势检测的 `GestureDetector` `widget`、用于应用主题数据传递的`Theme`等等

![Flutter框架](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/WidgetWork.png?x-oss-process=style/titanjun)


## Widget与Element

- `Widget`实际上就是`Element`的配置数据, `Widget`的功能是描述一个UI元素的一个配置数据, 而真正的UI渲染是由`Element`构成
- 由于`Element`是通过`Widget`生成，所以它们之间有对应关系，所以在大多数场景，我们可以宽泛地认为`Widget`就是指UI控件或UI渲染
- 一个`Widget`对象可以对应多个`Element`对象。这很好理解，根据同一份配置（`Widget`），可以创建多个实例（`Element`）

### `Widget`类的声明


```dart
@immutable
abstract class Widget extends DiagnosticableTree {
  /// Initializes [key] for subclasses.
  const Widget({ this.key });

  /// See also the discussions at [Key] and [GlobalKey].
  final Key key;

  /// multiple times.
  @protected
  Element createElement();

  /// A short, textual description of this widget.
  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

- `Widget`类继承自`DiagnosticableTree`，主要作用是提供调试信息。
- `Key`: 这个`key`属性类似于`React/Vue`中的`key`，主要的作用是决定是否在下一次`build`时复用旧的`widget`，决定的条件在`canUpdate()`方法中
- `createElement()`：正如前文所述`一个Widget可以对应多个Element`；`Flutter Framework`在构建UI时，会先调用此方法生成对应节点的`Element`对象。此方法是`Flutter Framework`隐式调用的，在我们开发过程中基本不会调用到。
- `debugFillProperties` 复写父类的方法，主要是设置`DiagnosticableTree`的一些特性。
- `canUpdate`是一个静态方法，它主要用于在`Widget`树重新`build`时复用旧的`widget`
    - 具体来说就是：是否用新的`Widget`对象去更新旧UI上所对应的`Element`对象的配置；
    - 通过其源码我们可以看到，只要`newWidget`与`oldWidget`的`runtimeType`和`key`同时相等时就会用`newWidget`去更新`Element`对象的配置，否则就会创建新的`Element`



### StatelessWidget和StatefulWidget的区别

- `StatelessWidget`是状态不可变的`widget`, 初始状态设置以后就不可再变化, 如果需要变化需要重新创建; `StatefulWidget`可以保存自己的状态
- 在`Flutter`中通过引入`State`来保存状态, 当`State`的状态改变时，能重新构建本节点以及孩子的`Widget`树来进行UI变化
- 如果需要主动改变`State`的状态，需要通过`setState()`方法进行触发，单纯改变数据是不会引发UI改变的
- 下面介绍部分的`Widget`组件



## Text

UI上面文字的展示基本上都要靠`Text`组件来完成

```dart
// 两种构造函数
// 显示普通的文本
 const Text(this.data, {
    Key key,
    this.style,
    this.textAlign,
    this.textDirection,
    this.locale,
    this.softWrap,
    this.overflow,
    this.textScaleFactor,
    this.maxLines,
    this.semanticsLabel,
  }) : assert(data != null),
       textSpan = null,
       super(key: key);

  /// 段落式文本,可以给文本中的每个textSpan设置其样式
  const Text.rich(this.textSpan, {
    Key key,
    this.style,
    this.textAlign,
    this.textDirection,
    this.locale,
    this.softWrap,
    this.overflow,
    this.textScaleFactor,
    this.maxLines,
    this.semanticsLabel,
  }): assert(textSpan != null),
      data = null,
      super(key: key);

```

### 参数介绍

#### data

文本的内容

```dart
Text('titanjun')
```

#### style

文本的样式

```dart
  const TextStyle({
    this.inherit = true,
    this.color,
    this.fontSize,
    this.fontWeight,
    this.fontStyle,
    this.letterSpacing,
    this.wordSpacing,
    this.textBaseline,
    this.height,
    this.locale,
    this.foreground,
    this.background,
    this.shadows,
    this.decoration,
    this.decorationColor,
    this.decorationStyle,
    this.debugLabel,
    String fontFamily,
    String package,
  }) : fontFamily = package == null ? fontFamily : 'packages/$package/$fontFamily',
       assert(inherit != null),
       assert(color == null || foreground == null, _kColorForegroundWarning);
       
// 相关属性介绍
1. inherit: 为false时不显示

2. color: 字体颜色

3. fontSize: 字体大小, 默认是14.0

4. fontWeight: 字体的粗体, FontWeight.w500
  
5. fontStyle: 字体的样式
    normal正常 italic 斜体

6. letterSpacing: 字符间距

7. wordSpacing: 单词间距

8. textBaseline
    alphabetic：用于对齐字母字符底部的水平线
    ideographic：用于对齐表意字符的水平线
    
9. height: 用在Text控件上的时候，会乘以fontSize做为行高,

10. locale: 国际化

11. foreground: 用paint来渲染text，也可以用他来改变字体颜色等

12. background: 背景颜色

13. decoration: 
    下划线 underline、 删除线 lineThrough、上划线 overline，默认是无 none
    
14. decorationStyle: decoration线的样式
    solid: 直线, double: 两条线, dotted: 短虚线, dashed: 长虚线, wavy: 波浪线

15. decorationColor: decoration线的颜色

16. debugLabel: 文本样式的描述, 该属性只在调试中维护

17. fontFamily和package（自定义字体的时候用的到，后面再详解）
```

使用样式示例

```dart
style: TextStyle(
    inherit: true,
    color: Colors.red,
    fontSize: 50,
    fontWeight: FontWeight.bold,
    fontStyle: FontStyle.italic,
    letterSpacing: 2,
    wordSpacing: 5,
    textBaseline: TextBaseline.alphabetic,
    height: 2,
    locale: Locale('CH'),
    decoration: TextDecoration.lineThrough,
    decorationColor: Colors.blue,
    decorationStyle: TextDecorationStyle.wavy,
),
```

#### textAlign

文本显示方向
```
left: 居左显示
center: 居中显示
right: 居右显示
justify: 文本的拉伸行，其末尾用软换行符填充宽度
start: 对齐容器前缘的文本。
    对于从左到右的文本([TextDirection.ltr])，这是左边缘。
    对于从右到左的文本([TextDirection.rtl])，这是右边缘。
end: 对齐容器尾部边缘的文本。
    对于从左到右的文本([TextDirection.ltr])，这是右边缘。
    对于从右到左的文本([TextDirection.rtl])，这是左边缘。
```

#### textDirection

和上述`TextAlign.start和TextAlign.end`一样

#### softWrap

文本是否能换行,bool类型

#### overflow

用来指定超出文本的表示方式，是截断文本啊还是用三个点显示等

```
ellipsis: ...形式显示
clip: 直接截断
fade: 效果和clip一样
```

#### maxLines

用来指定文本最多显示多少行

#### textScaleFactor

文本字体的缩放倍数，如：1.5则在默认字体上变成1.5倍大小字体，0.5则是0.5倍


### Text构造函数


```dart
child: Text(
      // 需要显示的文字
      'titanjun.top' * 3,
      textAlign: TextAlign.left,
      textDirection: TextDirection.ltr,
      locale: Locale('CH'),
      maxLines: 1,
      overflow: TextOverflow.fade,
      style: TextStyle(
          inherit: true,
          color: Colors.red,
          fontSize: 50,
          fontWeight: FontWeight.bold,
          fontStyle: FontStyle.italic,
          letterSpacing: 2,
          wordSpacing: 5,
          textBaseline: TextBaseline.alphabetic,
          height: 2,
          locale: Locale('CH'),
          decoration: TextDecoration.lineThrough,
          decorationColor: Colors.blue,
          decorationStyle: TextDecorationStyle.wavy,
      ),
  ),
```


### Text.rich构造函数

这个构造函数和`iOS`中用到的富文本类似

```dart
child: Text.rich(
    TextSpan(
      text: '博客地址: ',
      children: [
        TextSpan(
          text: 'https://',
          style: TextStyle(color: Colors.red)
        ),
        TextSpan(
          text: 'titanjun.top',
          style: TextStyle(color: Colors.blue),
        ),
        TextSpan(
          text: '欢迎访问',
          style: TextStyle(color: Colors.orange)
        ),
      ]
    ),
),
```


![Text.rich](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/text_rich.png)


其中`TextSpan`的构造函数如下

```dart
const TextSpan({
    this.style,
    this.text,
    // 接受List<TextSpan>类型的数组
    this.children,
    // 文本的手势操作, 后面说这个
    this.recognizer,
});
```

## Image

- 一个用于展示图片的组件。支持 JPEG、PNG、GIF、Animated GIF、WebP、Animated WebP、BMP 和 WBMP 等格式
- `Image`共有五种构造函数

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/ImageProvider.png)

### Image()

```dart
  const Image({
    Key key,
    // 一个图片对象ImageProvider, 可设置NetworkImage(), FileImage(), MemoryImage()三种对象
    @required this.image,
    // 图片的描述, String
    this.semanticLabel,
    this.excludeFromSemantics = false,
    // 图片的宽度, double
    this.width,
    // 图片的高度, double
    this.height,
    // 图像的颜色, 用于和图片混合的颜色, 结合colorBlendMode使用
    this.color,
    // 颜色和图片混合的状态, BlendMode
    this.colorBlendMode,
    // 图像在布局中分配的空间, BoxFit
    this.fit,
    // 图像边界内对齐图像, Alignment
    this.alignment = Alignment.center,
    // 未充分填充容器时,是否重复显示图片
    this.repeat = ImageRepeat.noRepeat,
    // 九片图像的中心切点, Rect
    this.centerSlice,
    // 是否在图像的方向上绘制图像 TextDirection
    this.matchTextDirection = false,
    // 当图像提供者发生变化时，是继续显示旧图像（true）还是暂时不显示（false）
    this.gaplessPlayback = false,
    // 设置图片的过滤质量
    this.filterQuality = FilterQuality.low,
  }) 
```

#### 部分属性详解

##### fit

图像在布局中分配的空间, `BoxFit`枚举值

- `fill`: 填充满容器空间, 图片会被拉伸
- `contain`: 以容器的大小等比例缩放图片
- `cover`: 填充整个容器, 图片会被剪切
- `fitWidth`: 以容器的宽度, 等比例缩放图片
- `fitHeight`: 以容器的高度, 等比例的缩放图片
- `none`: 以图片的实际大小显示
- `scaleDown`: 居中显示, 图片不会拉伸, 以宽高中最小的尺寸为标准


##### alignment

图像边界内对齐图像, `Alignment`类, 不是枚举值

```dart
  /// 定义方式为垂直方向-水平方向
  static const Alignment topLeft = Alignment(-1.0, -1.0);
  static const Alignment topCenter = Alignment(0.0, -1.0);
  static const Alignment topRight = Alignment(1.0, -1.0);
  static const Alignment centerLeft = Alignment(-1.0, 0.0);
  static const Alignment center = Alignment(0.0, 0.0);
  static const Alignment centerRight = Alignment(1.0, 0.0);
  static const Alignment bottomLeft = Alignment(-1.0, 1.0);
  static const Alignment bottomCenter = Alignment(0.0, 1.0);
  static const Alignment bottomRight = Alignment(1.0, 1.0);
  
  /// 使用方式
  alignment: Alignment.topLeft,
  // 或者
  alignment: Alignment(0.0, 1.0)
```


```dart
Image(
  image: NetworkImage('https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/flutter.jpeg?x-oss-process=style/titanjun'),
  fit: BoxFit.scaleDown,
  alignment: Alignment.topLeft,
),
```

### Image.network 

用于显示网络图片

```dart
Image.network(
  'https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/catimage.jpg',
  width: 100,
  height: 100,
  fit: BoxFit.scaleDown,
  alignment: Alignment.center,
)
```

网络请求`Image`是最常见的操作, 这里重点说明两个点

#### 缓存

- `ImageCache`是`ImageProvider`默认使用的图片缓存。`ImageCache`使用的是`LRU`的算法
- 默认可以存储1000张图片。如果觉得缓存太大，可以通过设置`ImageCache`的`maximumSize`属性来控制缓存图片的数量。
- 也可以通过设置`maximumSizeBytes`来控制缓存的大小（默认缓存大小10MB）


#### CDN优化

如果想要使用`cdn`优化，可以通过`url`增加后缀的方式实现。默认实现中没有这个点，但是考虑到`cdn`优化的可观收益，建议大家利用好这个优化



### Image.asset

- `Flutter`应用程序可以包含代码和 `assets`（有时称为资源）
- `asset`是打包到程序安装包中的，可在运行时访问
- 常见类型的`asset`包括静态数据（例如JSON文件），配置文件，图标和图片（JPEG，WebP，GIF，动画WebP / GIF，PNG，BMP和WBMP）
- `Flutter`使用`pubspec.yaml`文件（位于项目根目录），来识别应用程序所需的`asset`

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/image_assets.png)

<div class="note warning"><p>注意事项</p></div>

- 图片所在的文件夹`images`和`pubspec.yaml`需要在同一目录下, 否则`pubspec.yaml`文件中, 设置资源路径的时候要对应修改
- `images`图片文件夹中`2.0x和3.0x`图片要分别创建两个文件夹, 并把2倍和3倍图分别放在不同的文件夹中, 切文件的名字不要在带`@2x和@3x`字样

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/image_assets1.png)

```dart
Image.asset(
  'images/home.png',
  width: 100,
  height: 100,
  fit: BoxFit.scaleDown,
  alignment: Alignment.center,
)
```

### Image.file

```dart
Image.file(File file, {
    Key key,
    double scale = 1.0,
    this.semanticLabel,
    this.excludeFromSemantics = false,
    this.width,
    this.height,
    this.color,
    this.colorBlendMode,
    this.fit,
    this.alignment = Alignment.center,
    this.repeat = ImageRepeat.noRepeat,
    this.centerSlice,
    this.matchTextDirection = false,
    this.gaplessPlayback = false,
    this.filterQuality = FilterQuality.low,
  })
```


- 主要解析file参数，其他与`Image()`构造的参数一致！
- `file`: 对文件系统上的文件的引用。
- [File](https://docs.flutter.io/flutter/dart-io/File-class.html) 实例是一个对象，它包含可以在其上执行操作的路径


### Image.memory

```dart
Image.memory(Uint8List bytes, {
    Key key,
    double scale = 1.0,
    this.semanticLabel,
    this.excludeFromSemantics = false,
    this.width,
    this.height,
    this.color,
    this.colorBlendMode,
    this.fit,
    this.alignment = Alignment.center,
    this.repeat = ImageRepeat.noRepeat,
    this.centerSlice,
    this.matchTextDirection = false,
    this.gaplessPlayback = false,
    this.filterQuality = FilterQuality.low,
  })
```

- 加载`Uint8List`资源图片
- 主要解析`bytes`参数，其他与`Image()`构造的参数一致！

## Icon

- `Flutter`中，`Icon`是类似于`web`开发中一样使用`iconfont`(字体图标)，它是将图标做成字体文件，然后通过指定不同的字符而显示不同的图片
- 在字体文件中，每一个字符都对应一个位码，而每一个位码对应一个显示字形，不同的字体就是指字形不同，即字符对应的字形是不同的
- 而在`iconfont`中，只是将位码对应的字形做成了图标，所以不同的字符最终就会渲染成不同的图标。
- 在`Flutter`中`iconfont`相较于图片的优势如下:
  - 体积小：可以减小安装包大小。
  - 矢量的：`iconfont`都是矢量图标，放大不会影响其清晰度。
  - 可以应用文本样式：可以像文本一样改变字体图标的颜色、大小对齐等。
  - 可以通过`TextSpan`和文本混用。

### 使用Material Design字体图标

`Flutter`默认包含了一套`Material Design`的字体图标，在`pubspec.yaml`文件中的配置如下

```yaml
flutter:
  uses-material-design: true
```

<div class="note warning"><p>如果设置成false, 则图片效果如下, 图片颜色为自己设置的颜色</p></div>

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/icon_false.png)


- `Material Design`所有图标可以在其官网查看：https://material.io/tools/icons/
- `Material Design`所有图标对应的字符编码可在[material-design-icons](https://github.com/google/material-design-icons/blob/master/iconfont/codepoints)中搜索查找


### 在Text中使用

下面看一个在`Text`中使用`iconfont`的示例

```dart
String iconStr = "";
// accessible: &#xE914; or 0xE914 or E914
iconStr += "\uE914";
// error: &#xE000; or 0xE000 or E000
iconStr += " \uE000";
// fingerprint: &#xE90D; or 0xE90D or E90D
iconStr += " \uE90D";


Text(iconStr,
  style: TextStyle(
    fontFamily: "MaterialIcons",
    fontSize: 80.0,
    color: Colors.green
  ),
)
```

上述代码的运行效果如下

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/icon_image.png)

> 任何一个图片我们都可以使用`Text`文本进行展示, 但是这需要我们提供每一个图标的字符码点, 可在[material-design-icons](https://github.com/google/material-design-icons/blob/master/iconfont/codepoints)中搜索查找, 而且并不能固定指定图片的大小, 只能设置字体的大小, 这并对开发者不友好




### Icon介绍

`Flutter`封装了一个`Icon`来专门显示字体图标，上面的例子也可以用如下方式实现

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    Icon(Icons.accessible, color: Colors.green, size: 80),
    Icon(Icons.error, color: Colors.green, size: 80),
    Icon(Icons.fingerprint, color: Colors.green, size: 80),
  ],
)
```

`Icons`类中包含了所有`Material Design`图标的`IconData`静态变量定义, .....我大概算了一下, `Icons`中大概一共定义了984中图标

```dart
// icon的构造函数
const Icon(this.icon, {
    Key key,
    // double, 设置图片的大小, 同事设置宽高
    this.size,
    // Color, 设置图片的颜色
    this.color,
    // String, 图标的语义标签
    this.semanticLabel,
    // TextDirection, 从左/右开始排列
    this.textDirection,
})
```

## 按钮

- `Flutter`提供了`RaisedButton`、`FlatButton`、`OutlineButton`和`IconButton`四种按钮, 除了`IconButton`之外都是继承自`MaterialButton`
- 所有`Material`库中的按钮都有如下相同点：
  - 按下时都会有“水波动画”。
  - 有一个`onPressed`属性来设置点击回调，当按钮按下时会执行该回调，如果不提供该回调则按钮会处于禁用状态，禁用状态不响应用户点击


### MaterialButton

`MaterialButton`是除`IconButton`按钮之外的其他按钮的父类, 下面介绍一下各属性的使用

```dart
const MaterialButton({
    Key key,
    // 点击事件
    @required this.onPressed,
    // 高亮状态变化回调，参数：是否高亮，按下时高亮，抬起不高亮
    this.onHighlightChanged,
    // 字体的主体
    this.textTheme,
    // 按钮文字颜色
    this.textColor,
    // 禁用状态下按钮字体颜色
    this.disabledTextColor,
    // 按钮背景颜色
    this.color,
    // 禁用状态下背景颜色
    this.disabledColor,
    // 高亮状态(按下时的背景颜色)
    this.highlightColor,
    // 按钮的水波纹的颜色
    this.splashColor,
    // 字体亮度
    this.colorBrightness,
    // 按钮底部阴影效果的偏移量, double
    this.elevation,
    // 高亮状态下, 按钮底部阴影效果的偏移量, double
    this.highlightElevation,
    // 禁用状态下, 按钮底部阴影效果的偏移量, double
    this.disabledElevation,
    // 内边距
    this.padding,
    // 按钮的形状
    this.shape,
    this.clipBehavior = Clip.none,
    this.materialTapTargetSize,
    this.animationDuration,
    // 按钮的最小宽度
    this.minWidth,
    // 按钮的高度
    this.height,
    // 子widget
    this.child,
})
```

#### onPressed

按钮触发时触发的函数，如果不设置此属性`Button`为不可用状态

```dart
onPressed: () => print('被点击了'),
```

#### textTheme

按钮字体的主题, 在`onPressed`不为空的时候才有效果

```dart
/// 三个取值
ButtonTextTheme.normal
ButtonTextTheme.accent
ButtonTextTheme.primary
```

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/button_theme.png)

#### colorBrightness

设置按钮的字体亮度, 取值分别是`Brightness.light`和`Brightness.darks`

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/button_.png)


#### padding

内边距，其接收值的类型是`EdgeInsetsGeometry`类型的，`EdgeInsetsGeometry`是一个抽象类, 只能使用其子类`EdgeInsets`来实现

```dart
padding: EdgeInsets.all(10)
```

#### shape

- 设置按钮的形状，其接收值是`ShapeBorder`类型，`ShapeBorder`也是一个抽象类
- `ShapeBorder`的子类中比较常用的几个如下所示
  - `BeveledRectangleBorder` 带斜角的长方形边框
  - `CircleBorder` 圆形边框
  - `RoundedRectangleBorder` 圆角矩形
  - `StadiumBorder` 两端是半圆的边框

```dart
// 带斜角的长方形边框
const BeveledRectangleBorder({
    // 边框的样式 
    this.side = BorderSide.none,
    // 圆角大小
    this.borderRadius = BorderRadius.zero,
})

// 圆形边框
const CircleBorder({ 
    this.side = BorderSide.none 
})

// 圆角矩形
const RoundedRectangleBorder({
    this.side = BorderSide.none,
    this.borderRadius = BorderRadius.zero,
})

// 两端是半圆的边框
const StadiumBorder({ 
    this.side = BorderSide.none 
})

// 边框样式的设置
const BorderSide({
    // 边框颜色, 默认黑色 
    this.color = const Color(0xFF000000),
    // 边框宽度, 默认1.0
    this.width = 1.0,
    // 边框样式, solid: 实线边框(默认值), none: 不显示边框
    this.style = BorderStyle.solid,
})
```

下面就来看一下`shape`的配置和使用, 设置默认状态(即所有的边框样式和圆角都是默认值)的效果如下

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/button_shape1.png)

```dart
children: <Widget>[
  // BeveledRectangleBorder
  RaisedButton(
    child: Text('BeveledRectangleBorder'),
    onPressed: () => print('RaisedButton'),
    shape: BeveledRectangleBorder(
      borderRadius: BorderRadius.all(Radius.circular(10)),
      side: BorderSide(
        color: Colors.red,
        width: 2,
        style: BorderStyle.solid
      )
    ),
  ),
  RaisedButton(
    child: Icon(Icons.supervisor_account, color: Colors.green, size: 40),
    onPressed: () => print('RaisedButton'),
    padding: EdgeInsets.all(10),
    shape: CircleBorder(
      side: BorderSide(
        color: Colors.red,
        width: 2,
        style: BorderStyle.solid
      )
    ),
  ),
  RaisedButton(
    child: Text('RoundedRectangleBorder'),
    onPressed: () => print('RaisedButton'),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.all(Radius.circular(10)),
      side: BorderSide(
        color: Colors.red,
        width: 2,
        style: BorderStyle.solid
      )
    ),
  ),
  RaisedButton(
    child: Text('StadiumBorder'),
    onPressed: () => print('RaisedButton'),
    shape: StadiumBorder(
      side: BorderSide(
        color: Colors.red,
        width: 2,
        style: BorderStyle.solid
      )
    ),
  ),
],
```

上述代码是分别设置圆角和边框后的代码, 效果如下

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/button_shape2.png)


### RaisedButton

- `RaisedButton`即"漂浮"按钮，它默认带有阴影和灰色背景。按下后阴影会变大
- `RaisedButton`继承自`MaterialButton`, 相关属性和父类一样

```dart
RaisedButton(
    child: Text('RaisedButton'),
    onPressed: () => print('RaisedButton'),
)
```

### FlatButton

- `FlatButton`即扁平按钮，默认背景透明并不带阴影。按下后，会有背景色
- `RaisedButton`继承自`MaterialButton`, 相关属性和父类一样

```dart
FlatButton(
    child: Text('FlatButton'),
    onPressed: () => print('FlatButton'),
)
```


### OutlineButton

- `OutlineButton`默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和阴影(较弱)
- `RaisedButton`继承自`MaterialButton`, 相关属性和父类一样

```dart
OutlineButton(
    child: Text('OutlineButton'),
    onPressed: () => print('OutlineButton'),
)
```

### IconButton

`IconButton`是一个可点击的Icon，不包括文字，默认没有背景，点击后会出现背景

```dart
// 继承自StatelessWidget
class IconButton extends StatelessWidget {
  const IconButton({
    Key key,
    // 按钮的大小
    this.iconSize = 24.0,
    // 内边距
    this.padding = const EdgeInsets.all(8.0),
    // 按钮中图片的对齐方式
    this.alignment = Alignment.center,
    // 图片按钮的图片Icon
    @required this.icon,
    // 背景色
    this.color,
    // 高亮状态下的背景色
    this.highlightColor,
    // 按钮按下时, 水波纹的颜色
    this.splashColor,
    // 禁用状态下, 按钮的背景色
    this.disabledColor,
    // 点击事件
    @required this.onPressed,
    // String, 描述按钮按下时的描述文本, 需要长按才能出现(黑框显示文本)
    this.tooltip
  })
}
```

使用示例

```dart
IconButton(
    icon: Icon(Icons.mail_outline, color:Colors.orange, size: 40),
    color: Colors.yellow,
    iconSize: 100,
    alignment: Alignment.topLeft,
    onPressed: () => print('IconButton'),
    tooltip: 'titanjun.top',
)
```

### 图文按钮

每一个继承自`MaterialButton`的按钮`Widget`都有一个工厂构造函数, 返回一个图片在左, 文字在右的按钮

```dart
factory RaisedButton.icon({
    // 这里包含MaterialButton的所有属性
    ....
    // 图片Widget
    @required Widget icon,
    // 文字Widget
    @required Widget label,
  })
  
factory FlatButton.icon({
    ....
    @required Widget icon,
    @required Widget label,
  })
  
factory OutlineButton.icon({
    ....
    @required Widget icon,
    @required Widget label,
  })
  
// 使用示例
RaisedButton.icon(
  label: Text('data'),
  icon: Icon(Icons.mail),
  onPressed: () => {},
),
```


## 单选开关和复选框

- `Material widgets`库中提供了`Material`风格的单选开关`Switch`和复选框`Checkbox`，它们都是继承自`StatelessWidget`
- 它们本身不会保存当前选择状态，所以一般都是在父`widget`中管理选中状态
- 当用户点击`Switch`或`Checkbox`时，它们会触发`onChanged`回调，我们可以在此回调中处理选中状态改变逻辑

```dart
// Switch属性
const Switch({
    Key key,
    // Switch的状态值, true开启, false关闭
    @required this.value,
    // Switch改变状态所执行的操作
    @required this.onChanged,
    // 开启状态下选项条的颜色
    this.activeColor,
    // 开启状态下圆球的颜色
    this.activeTrackColor,
    // 关闭状态下选项条的颜色
    this.inactiveThumbColor,
    // 关闭状态下圆球的颜色
    this.inactiveTrackColor,
    // 设置开启状态下圆球的图片
    this.activeThumbImage,
    // 设置关闭状态下圆球的图片
    this.inactiveThumbImage,
    // 设置Switch的尺寸样式, padded: 建议大小48, shrinkWrap: 可能的最小尺寸
    this.materialTapTargetSize,
})

// Checkbox属性
const Checkbox({
    Key key,
    // Switch的状态值, true选中, false未选中
    @required this.value,
    // 如果为 true，那么复选框的值可以是 true，false 或 null
    // 如果为false(默认值), 那么只有true和false两种状态
    this.tristate = false,
    // 改变状态时执行的函数
    @required this.onChanged,
    // 选中时的颜色
    this.activeColor,
    // 设置Checkbox的尺寸样式, padded: 建议大小48, shrinkWrap: 可能的最小尺寸
    this.materialTapTargetSize,
})
```

使用代码

```dart
children: <Widget>[
  Switch(
    value: false,
    onChanged: (value) {},
    activeColor: Colors.red,
    activeTrackColor: Colors.yellow,
    inactiveThumbColor: Colors.blue,
    inactiveTrackColor: Colors.cyan,
    materialTapTargetSize: MaterialTapTargetSize.shrinkWrap
  ),
  Checkbox(
    value: true,
    onChanged: (value) { },
    activeColor: Colors.orange,
    tristate: true,
  )
],
```

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/widget_switch.png)



## 参考文档

- [Flutter中文网](https://book.flutterchina.club/chapter3/text.html)
- [Widgets 目录](https://flutterchina.club/widgets/)


---
