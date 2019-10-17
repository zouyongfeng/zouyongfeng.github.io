

![Flutter](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/flutter_layout.png?x-oss-process=style/titanjun)



<!--more-->


- 相关博客系列文章: [Flutter和Dart系列文章](https://www.titanjun.top/categories/Flutter%E7%AC%94%E8%AE%B0/)
- 相关`Demo`地址: [GitHub地址](https://github.com/CoderTitan/Flutter_Widget)
- 布局类`Widget`都会包含一个或多个子`widget`，不同的布局类`Widget`对子`widget`排版(`layout`)方式不同
- [上一篇文章](https://www.titanjun.top/Flutter%E4%B9%8BText%E5%92%8CImage.html)中提到: `Widget`实际上就是`Element`的配置数据, `Widget`的功能是描述一个`UI`元素的一个配置数据, 而真正的`UI`渲染是由`Element`构成
- 在`Flutter`中，根据`Widget`是否需要包含子节点将`Widget`分为了三类，分别对应三种`Element`，如下表


Widget | 对应的Element | 用途
--|--|--
`LeafRenderObjectWidget` | `LeafRenderObjectElement` | `Widget`树的叶子节点，用于没有子节点的`widget`，通常基础`widget`都属于这一类，如`Text`、`Image`
`SingleChildRenderObjectWidget` | `SingleChildRenderObjectElement` | 包含一个子`Widget`，如：`ConstrainedBox`、`DecoratedBox`等
`MultiChildRenderObjectWidget` | `MultiChildRenderObjectElement` | 包含多个子`Widget`，一般都有一个`children`参数，接受一个`Widget`数组。如`Row`、`Column`、`Stack`等


## 布局类Widget

- 布局类`Widget`就是指直接或间接继承(包含)`MultiChildRenderObjectWidget`的`Widget`，它们一般都会有一个`children`属性用于接收子`Widget`
- `Widget`的继承关系如下:
  - `Widget` > `RenderObjectWidget` > `(Leaf/SingleChild/MultiChild)RenderObjectWidget`
- `RenderObjectWidget`类中定义了创建、更新`RenderObject`的方法，子类必须实现他们
- 对于布局类`Widget`来说，其布局算法都是通过对应的`RenderObject`对象来实现的
- `Flutter`中主要有以下几种布局类的`Widget`：
  - 线性布局`Row`和`Column`
  - 弹性布局`Flex`
  - 流式布局`Wrap`、`Flow`
  - 层叠布局`Stack`、`Positioned`


## 线性布局

- `Row`和`Column`是一种现行布局的`Widget`, 都继承自`Flex`
- 所谓线性布局，即指沿水平或垂直方向排布`子Widget`
- 对于线性布局，有主轴和纵轴之分，如果布局是沿水平方，那么主轴就指是水平方向，而纵轴即垂直方向；如果布局沿垂直方向，那么主轴就是指垂直方向，而纵轴就是水平方向
- `Row`的主轴即为水平方向, `Column`的主轴是垂直方向, 切两者的属性和使用都一样
- 相关下定义的源码如下:


```dart
Row({
    Key key,
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
    MainAxisSize mainAxisSize = MainAxisSize.max,
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
    TextDirection textDirection,
    VerticalDirection verticalDirection = VerticalDirection.down,
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
})

Column({
    Key key,
    MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
    MainAxisSize mainAxisSize = MainAxisSize.max,
    CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
    TextDirection textDirection,
    VerticalDirection verticalDirection = VerticalDirection.down,
    TextBaseline textBaseline,
    List<Widget> children = const <Widget>[],
})  
```

### 相关属性如下

#### `mainAxisAlignment`

子`Widget`在主轴方向的排列方式, 为方便以下皆称`Widget`为组件

```dart
// 默认值
MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start
```

- `start`: 子`widgets`向主轴起点对其, 依次排列
- `end`: 子`widgets`向主轴终点对其, 依次排列
- `center`: 所有子`widgets`居中排列
- `spaceBetween`: 均匀分配,相邻`widgets`间距离相同。每行第一个`widgets`与行首对齐，每行最后一个`widgets`与行尾对齐
- `spaceAround`: 均匀分配,相邻`widgets`间距离相同。每行第一个`widgets`到行首的距离和每行最后一个`widgets`到行尾的距离将会是相邻`widgets`之间距离的一半
- `spaceEvenly`: 均匀分配,相邻`widgets`间距离相同。每行第一个`widgets`到行首的距离和每行最后一个`widgets`到行尾的距离和相邻`widgets`之间距离相同


属性 | 效果
---|---
`start` | ![start](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_start.png)
`end` | ![end](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_end.png)
`center` | ![center](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_center.png)
`spaceBetween` | ![spaceBetween](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_between.png)
`spaceAround` | ![spaceAround](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_around.png)
`spaceEvenly` | ![spaceEvenly](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_evenly.png)


#### `mainAxisSize`

```dart
// 默认值
MainAxisSize mainAxisSize = MainAxisSize.max
```

- 表示`Row`在主轴(水平)方向占用的空间，默认是`MainAxisSize.max`
- `max`表示尽可能多的占用水平方向的空间，此时无论子`widgets`实际占用多少水平空间，`Row`的宽度始终等于水平方向的最大宽度；
- `MainAxisSize.min`表示尽可能少的占用水平空间，当子`widgets`没有占满水平剩余空间，则`Row`的实际宽度等于所有子`widgets`占用的的水平空间


#### `verticalDirection`

表示Row纵轴（垂直）的对齐方向, 默认值`down`，表示从上到下; `up`表示从下到上

```dart
// 默认值
VerticalDirection verticalDirection = VerticalDirection.down
```

#### `crossAxisAlignment`


- 表示子`Widgets`在纵轴方向的对齐方式，`Row`的高度等于子`Widgets`中最高的子元素高度
- `crossAxisAlignment`的参考系是`verticalDirection`

```dart
// 默认值
CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center

/**
 * VerticalDirection.down时, crossAxisAlignment.start指顶部对齐
 * VerticalDirection.up时，crossAxisAlignment.start指底部对齐
 * crossAxisAlignment.end和crossAxisAlignment.start正好相反
 */
```

- 当`VerticalDirection.down`时, `crossAxisAlignment`个枚举值如下
- `start`: 顶部对其
- `end`: 底部对其
- `center`: 居中对其
- `stretch`: 侧轴方向上, 子`Widget`的高度拉伸至和`Row`的高度相同
- `baseline`: 不论`VerticalDirection`取值如何, 子`Widget`的顶部和`Row`的顶部对其


#### `textDirection`

表示水平方向子widget的布局顺序(是从左往右还是从右往左)，默认为系统当前Locale环境的文本方向(如中文、英语都是从左往右，而阿拉伯语是从右往左)

```dart
TextDirection textDirection
/**
 * ltr: 从左往右
 * rtl: 从右往左
 */
```


#### `textBaseline`

用于对其文本的水平线, [详情可参考](http://www.runoob.com/tags/canvas-textbaseline.html)

```dart
TextBaseline textBaseline
/**
 * alphabetic: 用于对齐普通的字母基线
 * ideographic: 用于对齐表意基线
 */
```

### 使用代码

```dart
    Row(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.center,
        textDirection: TextDirection.ltr,
        verticalDirection: VerticalDirection.down,
        textBaseline: TextBaseline.ideographic,
        children: <Widget>[
          new Container(width: 80.0, height:80.0, color: Colors.red,),
          new Container(width: 80.0, height:90.0, color: Colors.green,),
          new Container(width: 80.0, height:100.0, color: Colors.blue,),
        ],
    )
```


<div class="note warning"><p>特别注意</p></div>

在`Row`和`Column`中, 如果子`widget`超出屏幕范围，则会报溢出错误

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/row_column.png)


## 弹性布局

- 弹性布局允许子`widget`按照一定比例来分配父容器空间
- `Flutter`中的弹性布局主要通过`Flex`和`Expanded`来配合实现
- `Flex`可以沿着水平或垂直方向排列子`widget`
- 如果已知主轴方向，建议使用`Row`或`Column`，因为`Row`和`Column`都继承自`Flex`，参数基本相同，所以能使用`Flex`的地方一定可以使用`Row`或`Column`
- `Flex`本身功能是很强大的，它也可以和`Expanded`配合实现弹性布局，接下来我们只讨论`Flex`和弹性布局相关的属性(其它属性已经在介绍`Row`和`Column`时介绍过了)

### Flex

```dart
Flex({
    Key key,
    //弹性布局的方向
    @required this.direction,
    this.mainAxisAlignment = MainAxisAlignment.start,
    this.mainAxisSize = MainAxisSize.max,
    this.crossAxisAlignment = CrossAxisAlignment.center,
    this.textDirection,
    this.verticalDirection = VerticalDirection.down,
    this.textBaseline,
    List<Widget> children = const <Widget>[],
})

// direction
// 水平方向
Axis direction = Axis.horizontal
// 垂直方向, 默认为垂直方向
Axis direction = Axis.vertical
```

> `Flex`继承自`MultiChildRenderObjectWidget`，对应的`RenderObject`为`RenderFlex`，`RenderFlex`中实现了其布局算法



### Expanded

可以按比例缩放`Row`、`Column`和`Flex`子`widget`所占用的空间

```dart
class Expanded extends Flexible {
  
  const Expanded({
    Key key,
    int flex = 1,
    @required Widget child,
  }) : super(key: key, flex: flex, fit: FlexFit.tight, child: child);
}
```

> `flex`为弹性系数，如果为0或`null`，则`child`是没有弹性的，即不会被扩伸占用的空间
> 如果大于0，所有的`Expanded`按照其`flex`的比例来分割主轴的全部空闲空间


```dart
    Row(
        children: <Widget>[
          Container(width: 80.0, height:80.0, color: Colors.red,),
          Expanded(
            flex: 1,
            child: Container(width: 80.0, height:80.0, color: Colors.blue,),
          ),
          Expanded(
            flex: 1,
            child: Container(width: 80.0, height:80.0, color: Colors.yellow,),
          )
        ],
      ),
```


## 流式布局

- 上面提到在`Row`和`Column`中, 如果子`widget`超出屏幕范围，则会报溢出错误
- 这是因为`Row`默认只有一行，如果超出屏幕不会折行
- 我们把超出屏幕显示范围会自动折行的布局称为流式布局
- `Flutter`中通过`Wrap`和`Flow`来支持流式布局

### Wrap

```dart
Wrap({
    Key key,
    this.direction = Axis.horizontal,
    this.alignment = WrapAlignment.start,
    this.spacing = 0.0,
    this.runAlignment = WrapAlignment.start,
    this.runSpacing = 0.0,
    this.crossAxisAlignment = WrapCrossAlignment.start,
    this.textDirection,
    this.verticalDirection = VerticalDirection.down,
    List<Widget> children = const <Widget>[],
})
```

可以看到`Wrap`中的很多属性和`Row`中相同, 这里就不在赘述了, 这里主要看一下`Wrap`中特有的属性

#### alignment

子`Widget`在主轴上的对其方式

```dart
// 默认值
this.alignment = WrapAlignment.start
// 取值: start, end, center, spaceBetween, spaceAround, spaceEvenly
```

#### runAlignment

子`Widget`在纵轴上的对其方式

```dart
// 默认值
this.runAlignment = WrapAlignment.start
// 取值: start, end, center, spaceBetween, spaceAround, spaceEvenly
```

#### spacing

主轴方向子`widget`的间距: `spacing: 10`

#### runSpacing

纵轴方向子`widget`的间距: `runSpacing: 10`


### Flow

- 一般很少会使用`Flow`，因为其过于复杂，需要自己实现子`widget`的位置转换，在很多场景下首先要考虑的是`Wrap`是否满足需求
- `Flow`主要用于一些需要自定义布局的UI或性能要求较高(如动画中)的场景
- `Flow`有如下优点：
  - 性能好: `Flow`是一个对`child`尺寸以及位置调整非常高效的控件，`Flow`用转换矩阵对`child`进行位置调整的时候进行了优化
  - 在`Flow`定位过后，如果`child`的尺寸或者位置发生了变化，在`FlowDelegate`中的`paintChildren()`方法中调用`context.paintChild` 进行重绘，而`context.paintChild`在重绘时使用了转换矩阵，并没有实际调整`Widget`位置。
  - 灵活: 由于我们需要自己实现`FlowDelegate`的`paintChildren()`方法，所以我们需要自己计算每一个`widget`的位置，因此，可以实现自定义布局。
- 缺点：
  - 使用复杂.
  - 不能自适应子`widget`大小，必须通过指定父容器大小或重写`FlowDelegate`的`getSize`返回固定大小
- 下面是一个简单的示例代码:


```dart
class FlowWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Container(
      color: Colors.orange,
      child: Flow(
        delegate: ShowFlowDelegate(margin: EdgeInsets.all(10)),
        children: <Widget>[
          Container(width: 100.0, height:100.0, color: Colors.red),
          Container(width: 100.0, height:100.0, color: Colors.yellow),
          Container(width: 100.0, height:100.0, color: Colors.blue),
          Container(width: 100.0, height:100.0, color: Colors.cyan),
          Container(width: 100.0, height:100.0, color: Colors.pink)
        ],
      ),
    );
  }
}
```

实现一个继承自`FlowDelegate`的类, 并重写响应的方法

```dart
class ShowFlowDelegate extends FlowDelegate {
  EdgeInsets margin =EdgeInsets.zero;
  ShowFlowDelegate({this.margin});

  @override
  void paintChildren(FlowPaintingContext context) {
    var x = margin.left;
    var y = margin.top;
    //计算每一个子widget的位置  
    for (int i = 0; i < context.childCount; i++) {
      var w = context.getChildSize(i).width + x + margin.right;
      if (w < context.size.width) {
        context.paintChild(i,
            transform: new Matrix4.translationValues(
                x, y, 0.0));
        x = w + margin.left;
      } else {
        x = margin.left;
        y += context.getChildSize(i).height + margin.top + margin.bottom;
        //绘制子widget(有优化)  
        context.paintChild(i,
            transform: new Matrix4.translationValues(
                x, y, 0.0));
         x += context.getChildSize(i).width + margin.left + margin.right;
      }
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    // 设置Flow的大小
    return Size(double.infinity, 300);
  }

  @override
  bool shouldRepaint(FlowDelegate oldDelegate) {
    return oldDelegate !=this;
  }
}
```


## 层叠布局

- 层叠布局和`Web`中的绝对定位、`iOS`中的`Frame`布局是相似的，子`widget`可以根据到父容器四个角的位置来确定本身的位置
- 绝对定位允许子`widget`堆叠（按照代码中声明的顺序）
- `Flutter`中使用`Stack`和`Positioned`来实现绝对定位，`Stack`允许子`widget`堆叠，而`Positioned`可以给子`widget`定位

### Stack

```dart
Stack({
  Key key,
  this.alignment = AlignmentDirectional.topStart,
  this.textDirection,
  this.fit = StackFit.loose,
  this.overflow = Overflow.clip,
  List<Widget> children = const <Widget>[],
})
```

#### alignment

决定子`Widget`在`Stack`中的定位

```dart
// 默认值
this.alignment = AlignmentDirectional.topStart

// 取值如下, start和end为水平方向, top和bottom是垂直方向
static const AlignmentDirectional topStart = AlignmentDirectional(-1.0, -1.0);
static const AlignmentDirectional topCenter = AlignmentDirectional(0.0, -1.0);
static const AlignmentDirectional topEnd = AlignmentDirectional(1.0, -1.0);

static const AlignmentDirectional centerStart = AlignmentDirectional(-1.0, 0.0);
static const AlignmentDirectional center = AlignmentDirectional(0.0, 0.0);
static const AlignmentDirectional centerEnd = AlignmentDirectional(1.0, 0.0);

static const AlignmentDirectional bottomStart = AlignmentDirectional(-1.0, 1.0);
static const AlignmentDirectional bottomCenter = AlignmentDirectional(0.0, 1.0);
static const AlignmentDirectional bottomEnd = AlignmentDirectional(1.0, 1.0);

// 还可以使用具体数值比例定位, 设置值在0~1之间
AlignmentDirectional(0.8, 0.9)
```

#### textDirection

决定`alignment`对齐的参考系

```dart
// 默认ltr
textDirection: TextDirection.ltr

// textDirection的值为TextDirection.ltr，则alignment的start代表左，end代表右
// textDirection的值为TextDirection.rtl，则alignment的start代表右，end代表左
```

#### fit

用于决定没有定位的子`widget`如何去适应`Stack`的大小

```dart
// 默认值
this.fit = StackFit.loose

// StackFit.loose表示使用子widget的大小
// StackFit.expand表示扩伸到Stack的大小
```

#### overflow

决定如何显示超出`Stack`显示空间的子`widget`

```dart
// 默认值
this.overflow = Overflow.clip

// Overflow.clip时，超出部分会被剪裁（隐藏)
// Overflow.visible时，时则不会被剪裁
```

#### 使用示例

```dart
Stack(
  // alignment: AlignmentDirectional.center,
  alignment: AlignmentDirectional(0.8, 0.8),
  textDirection: TextDirection.ltr,
  fit: StackFit.loose,
  overflow: Overflow.visible,
  children: <Widget>[
    Container(width: 100.0, height:100.0, color: Colors.red),
    Container(width: 100.0, height:100.0, color: Colors.yellow),
  ],
)
```

### Positioned

`Positioned`和`iOS`中的`Frame`设置位置和大小一样, 根据上下左右和宽高设置`Widget`的定位和大小

```dart
const Positioned({
    Key key,
    this.left,
    this.top,
    this.right,
    this.bottom,
    this.width,
    this.height,
    @required Widget child,
})
```

> `left、top 、right、 bottom`分别代表离Stack左、上、右、底四边的距离, `width`和`height`用于指定定位元素的宽度和高度

> 注意，此处的width、height 和其它地方的意义稍微有点区别，此处用于配合left、top 、right、 bottom来定位widget，举个例子，在水平方向时，你只能指定left、right、width三个属性中的两个，如指定left和width后，right会自动算出(left+width)，如果同时指定三个属性则会报错，垂直方向同理

```dart
child: Stack(
  alignment: AlignmentDirectional.center,
  children: <Widget>[
    // 这个widget会根据alignment的设置展示
    Container(child: Text('https', style: TextStyle(color: Colors.red)), color: Colors.yellow,),
    // 这个widget会根据left和top和width的设置显示和alignment无关了, 实际width为80
    Positioned(
      left: 10,
      top: 30,
      width: 80,
      child: Container(width: 100.0, height:100.0, color: Colors.red),
    ),
    Positioned(
      right: 10,
      bottom: 50,
      child: Container(width: 100.0, height:100.0, color: Colors.blue),
    )
  ],
),
```

> 至此, `Flutter`中布局相关的`Widget`也都学习完了......接下来就是容器类`Widget`了


## 参考文献
- [Flutter实战](https://book.flutterchina.club/chapter5/)
- [Flutter中文网](https://flutterchina.club/widgets/layout/)


