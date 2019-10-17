

![dart-logo](https://titanjun.oss-cn-hangzhou.aliyuncs.com/flutter/dart-logo.png?x-oss-process=style/titanjun)

<!--more-->

- `Dart`是谷歌在 2011 年推出的编程语言，是一种结构化`Web`编程语言，允许用户通过`Chromium`中所整合的虚拟机（`Dart VM`）直接运行`Dart` 语言编写的程序，免去了单独编译的步骤
- 以后这些程序将从`Dart VM`更快的性能与较低的启动延迟中受益
- `Dart`从设计之初就为配合现代`web`整体运作而考虑，开发团队也同时在持续改进`Dart`向`JavaScript`转换的快速编译器
- `Dart VM`以及现代`JavaScript`引擎（V8 等）都是`Dart`语言的首选目标平台
- `Dart`语言和`Swift`语言有很多的相似之处


## 重要概念

在学习`Dart`语言之前, 先了解一些`Dart`相关的一些概念:
- 在`O-Objective`中有一切皆对象的说法, 这句话在`Dart`中同样适用
  - 所有能够使用变量引用的都是对象， 每个对象都是一个类的实例
  - 在`Dart`中 甚至连 数字、方法和`null`都是对象
  - 所有的对象都继承于`Object`类
- `Dart`动态类型语言, 尽量给变量定义一个类型，会更安全，没有显示定义类型的变量在`debug`模式下会类型会是`dynamic`(动态的)
- `Dart`会在运行之前解析你的所有代码，指定数据类型和编译时的常量，可以提高运行速度
- `Dart`中的类和接口是统一的，类即接口，你可以继承一个类，也可以实现一个类（接口），自然也包含了良好的面向对象和并发编程的支持
- `Dart`函数
  - 支持顶级函数 (例如`main()`)
  - 支持在类中定义函数, 如静态函数和实例函数 
  - 还可以在方法中定义方法（嵌套方法或者局部方法）
- 类似的，`Dart`支持顶级变量，以及依赖于类或对象（静态变量和实例变量）变量。实例变量有时被称为域或属性
- `Dart`不具备关键字`public`，`protected`和`private`。如果一个标识符以下划线`（_）`开始，那么它和它的库都是私有的
- 标识符可以字母或（_）开始，或者是字符加数字的组合开头

> 以上只是大概说明了一些`Dart`中的重要概念, 具体的语法使用, 请看下文


## 基本语法

### 注释

`Dart`的注释分为3种：单行注释、多行注释、文档注释
- 单行注释以`//`开头
- 多行注释以`/*`开头，以`*/`结尾
- 文档注释以`///`或者`/**`开头


### 分号;

- 分号用于分隔`Dart`语句
- 通常我们在每条可执行的语句结尾添加分号
- 使用分号的另一用处是在一行中编写多条语句
- 在`Dart`中，用分号来结束语句是必须的, 不加分则会报错

### 其他语法

- 按照`Dart`的编程规范，使用2个空格来缩进
- 输出语句使用`print(Object)`


## 变量和常量

### 变量

使用`var`、`Object`或`dynamic`关键字声明变量

#### 弱类型变量

`var`方式声明变量

```dart
  // 1. 声明变量, 不赋初始值
  var a;
  a = 'titanjun';
  a = 123;
  a = false;
  print(a);
  // 在不初始化的前提下, 变量可以赋值任何类型的值


  // 2. 声明变量, 赋初始值
  var b = 'titanjun.top';
  // 变量在有初始化值得情况下, 只能赋值相同类型的值, 否则报错
  // b = 123;
```

`dynamic`声明

```dart
  dynamic c = 'titannjun';
  c = 123;
  c = false;
  print(c);
  // 调用未声明的方法, 不会报错, 运行时报错
  // c.test();
```

`Object`方式声明变量

```dart
  Object d = 'titanjun';
  d = 123;
  d = false;
  // 调用未声明的方法, 会直接报错
  // d.test();
```

#### 强类型变量

明确指定变量的类型, 声明后，类型被锁定

```dart
String str = 'titanjun';
bool isStr = true;
num number = 1234;
```

名称 | 	说明
--|--
`num` | 	数字
`int` | 		整型
`double` | 		浮点
`bool` | 		布尔
`String` | 		字符串
`StringBuffer` | 		字符串 buffer
`DateTime` | 		时间日期
`Duration` | 		时间区间
`List` | 		列表
`Sets` | 		无重复队列
`Maps` | 		kv 容器
`enum` | 		枚举


```dart
String a = 'doucafecat';
int i = 123;
double d = 0.12;
bool b = true;
DateTime dt = new DateTime.now();
List l = [ a, i, d, b, dt];
```

#### 默认值

一切都是`Object`, 未初始化的变量的初始值为`null`, 即使是数字也是如此，因为在`Dart`中数字也是一个对象

```dart
  bool isNum;
  String str2;
  StringBuffer str3;
  num num1;
  print([isNum, str2, str3, num1]);
  // [null, null, null, null]
```

> 可选类型

在声明变量的时候，你可以选择加上具体的类型：

```dart
String name2 = 'name2';
```

- 这种方式可以更加清晰的表达你想要定义的变量的类型, 编译器也可以根据该类型为你提供代码补全、提前发现 bug 等功能
- 注意: 对于局部变量，这里遵守 [代码风格推荐](http://dart.goodev.org/guides/language/effective-dart/design#type-annotations) 部分的建议，使用`var`而不是具体的类型来定义局部变量


#### 强弱类型

- 在写 API 接口的时候，请用`强类型`，一旦不符合约定，接收数据时能方便排查故障
- 写个小工具时，可以用`弱类型`，这样代码写起来很快，类型自动适应


### 常量

- 常量使用`final`或者`const`
- 一个`final`变量只能赋值一次
- 一个`const`变量是编译时常量
- 实例变量可以为`final`但是不能是`const`
- `final`不能和`var`同用
- `const`不能和`var`同用

#### final

`final`修饰的变量(即常量2)

```dart
  // 类型声明可以省略
  final age = 10;
  final int age1 = 20;

  // final修饰的变量不能重新赋值, 会报错
  age = 20;
  age1 = 30;
```

#### const

- `const`变量为编译时常量
- 如果在类中使用`const`定义常量，请定义为`static const`
- 使用`const`定义的常量, 可以直接给定初始值，也可以使用其他`const`变量的值来初始化其值

```dart
  // 类型声明可以省略
  const m1 = 12;
  const double m2 = 23;
  const m3 = m1 + m2;

  // final修饰的变量不能重新赋值, 会报错
  m1 = 10;
  m2 = 1.02;
```

#### final和const的区别

1. 需要确定值

```dart
// final是运行时的时候判断, const是赋值时进行判断
final time1 = DateTime.now();
// const修饰的常量会报错
const time2 = DateTime.now();
```

2. 不可变性可传递

```dart
  final List ls = [11, 22, 33];
  // final修饰的数组可以改变元素值
  ls[1] = 44;

  const List ls1 = [11, 22, 33];
  // const修饰的数组不可以改变元素值, 运行时报错
  ls1[1] = 44;
```

3. 内存中重复创建

```dart
  final arr1 = [11 , 22];
  final arr2 = [11 , 22];
  // 判断是否是相同内存
  print(identical(arr1, arr2));  // false

  // const修饰的常量, 在内存中不会重复创建相同的常量
  const ls3 = [11 , 22];
  const ls4 = [11 , 22];
  print(identical(ls3, ls4));  // true
```



## 操作符

### 算术操作符

`Dart`支持常用的算术操作符

操作符 |	解释
--|--
`+` |	加号
`–` |	减号
`-expr` |	负号
`*` |	乘号
`/` |	除号(值为`double`类型)
`~/` |	除号，但是返回值为整数
`%` |	取模

示例:

```dart
assert(2 + 3 == 5);
assert(2 - 3 == -1);
assert(2 * 3 == 6);
assert(5 / 2 == 2.5);   // 结果是double类型
assert(5 ~/ 2 == 2);    // 结果是integer类型 
assert(5 % 2 == 1);     // 余数
```

### 自加自减

`Dart`还支持自加自减操作
- `++var`: 先自加在使用
- `var++`: 先使用在自加
- `--var`: 先自减在使用
- `var--`: 先使用在自减

示例

```dart
  var a = 0, b = 0;
  
  b = a++;
  print('a = $a, b = $b'); //a = 1, b = 0
  b = ++a;
  print('a = $a, b = $b'); //a = 2, b = 2


  b = a--;
  print('a = $a, b = $b'); //a = 1, b = 2
  b = --a;
  print('a = $a, b = $b'); //a = 0, b = 0
```

### 关系操作符

运算符 |	含义
--|--
`==` | 	等于
`!=` | 	不等于
`>` | 	大于
`<` | 	小于
`>=` | 	大于等于
`<=` | 	小于等于

- 要测试两个对象代表的是否为同样的内容，使用 `==` 操作符
- 在某些情况下，你需要知道两个对象是否是同一个对象， 使用`identical()`方法


```dart
external bool identical(Object a, Object b);
```

### 类型判定操作符

类型判定操作符是在运行时判定对象类型的操作符

操作符 |	解释
--|--
`as` |	类型转换
`is` |	如果对象是指定的类型返回 True
`is!` | 	如果对象是指定的类型返回 False

- 只有当`obj`实现了 `T` 的接口， `obj is T` 才是`true`。例如`obj is Object`总是`true`
- 使用`as`操作符把对象转换为特定的类型
- 可以把`as`它当做用`is`判定类型然后调用 所判定对象的函数的缩写形式

```dart
if (emp is Person) { // Type check
  emp.firstName = 'Bob';
}

// 上面代码可简化为
(emp as Person).firstName = 'Bob';
```

<div class="note warning"><p>注意</p></div>

如果`emp`是`null`或者不是`Person`类型，则第一个示例使用`is`则不会执行条件里面的代码，而第二个情况使用`as`则会抛出一个异常; 所以在不缺定`emp`是否为空的情况下, 安全起见, 建议使用第一种方式


### 赋值操作符

  |  |  |  |  |  |  | 
--|--|--|--|--|--
`=`  | 	`–=` | `/=`  | `%=`  | 	`>>=` |  `^=`
`+=` | `*=`  | `~/=` | `<<=` | 	`&=`  |  


示例:

```dart
// 给 a 变量赋值
a = value;  

// 复合赋值操作符
a += b;  // 等价于a = a + b;

// 如果 b 是 null，则赋值给 b；
// 如果不是 null，则 b 的值保持不变
b ??= value; 
     
// 如下所示:     
  var s;
  print(s);  // null
  print(s ?? 'str');  // str
  s ??= 'string';
  print(s);  // string
```

### 逻辑操作符

可以使用逻辑操作符来 操作布尔值：

- `!expr`: 对表达式结果取反（true 变为 false ，false 变为 true）
- `||`: 逻辑 OR
- `&&`: 逻辑AND


### 条件表达式

```
condition ? expr1 : expr2
// 如果 condition 是 true，执行 expr1 (并返回执行的结果)； 否则执行 expr2 并返回其结果

expr1 ?? expr2
// 如果 expr1 是 non-null，返回其值； 否则执行 expr2 并返回其结果
```

示例:

```
String toString() => msg ?? super.toString();

// 上面的代码等价于
String toString() => msg == null ? super.toString() : msg;

// 等价于
String toString() {
  if (msg == null) {
    return super.toString();
  } else {
    return msg;
  }
}
```

### 级联操作符

级联操作符 (`..`) 可以在同一个对象上 连续调用多个函数以及访问成员变量。 使用级联操作符可以避免创建 临时变量， 并且写出来的代码看起来 更加流畅

```dart
querySelector('#button') // Get an object.
  ..text = 'Confirm'   // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

第一个方法`querySelector()`返回了一个`selector`对象。 后面的级联操作符都是调用这个对象的成员， 并忽略每个操作 所返回的值

```dart
// 上面代码等价于
var button = querySelector('#button');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

级联调用也可以嵌套：

```dart
final addressBook = (new AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (new PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

<div class="note warning"><p>注意</p></div>

严格来说，两个点的级联语法不是一个操作符, 只是一个`Dart`特殊语法。


## 流程控制语句

在`Dart`中可以使用下面的语句来控制`Dart`代码的流程：
- `if-else`
- `for`和`for-in`
- `while`和`do-while`
- `switch`
- `assert`
- `break`和`continue`
- `try-catch`和`throw`

### `if-else`

`Dart`支持`if`语句以及可选的`else`

```dart
  if (a == 0) {
    print('a = 0');
  } else if (a == 1) {
    print('a = 1');
  } else {
    print('a = 2');
  }
```

<div class="note warning"><p>注意</p></div>

上述代码中的条件控制语句的结果必须是布尔值

### for

可以使用标准的`for`循环, `List`和`Set`等实现了`Iterable`接口的类还支持`for-in`形式的遍历：

```dart
  var arr = [0, 1, 2];

  // for循环
  for (var i = 0; i < arr.length; i++) {
    print(arr[i]);
  }
  
  // for-in循环
  for (var x in arr) {
    print(x);
  }
```

### `While`和`do-while`


```dart
  // while 循环在执行循环之前先判断条件是否满足：
  while (c == 0) {
    print('c = $c');
  }

  // 而do-while循环是先执行循环代码再判断条件：
  do {
    print('c = $c');
  } while (c == 0);
```

### `Break`和`continue`

使用`break`来终止循环：

```dart
while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}
```

使用`continue`来开始下一次循环

```dart
for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}
```

### `Switch`

- `Dart`中的`Switch`语句使用 `==` 比较 `integer`、`string`、或者编译时常量
- 比较的对象必须都是同一个类的实例, 比较适合枚举值
- 每个非空的`case`语句都必须有一个`break`语句
- 另外还可以通过`continue`、`throw`或者`return`来结束非空`case`语句
- 当没有`case`语句匹配的时候，可以使用`default`语句来匹配这种默认情况
- 每个`case`语句可以有局部变量，局部变量只有在这个语句内可见

```dart
  var command = 'OPEN';
  switch (command) {
    case 'CLOSED':
      print('CLOSED');
      break;
    case 'APPROVED':
      print('APPROVED');
      // break;
      // 这里非空的case, 没有break会报错
    case 'DENIED':
      // 这里空的case, 可以不要break
    case 'OPEN':
      print('OPEN');
      continue nowClosed;
  //如果你需要实现这种继续到下一个 case 语句中继续执行，则可以 使用 continue 语句跳转到对应的标签（label）处继续执行：
  nowClosed:
    case 'PENDING':
      print('PENDING');
      break;
    default:
      print('default');
  }
```

### Assert

- 断言: 如果条件表达式结果不满足需要，则可以使用`assert` 语句中断代码的执行
- `assert`方法的参数可以为任何返回布尔值的表达式或者方法。
- 如果返回的值为`true`，断言执行通过，执行结束
- 如果返回值为`false`，断言执行失败，会抛出一个异常
- 可在开发过程中, 监测代码是否有问题时使用

```dart
// Make sure the variable has a non-null value
assert(text != null);

// Make sure the value is less than 100
assert(number < 100);

// Make sure this is an https URL
assert(urlString.startsWith('https'));
```

<div class="note warning"><p>注意</p></div>

断言只在开发模式下运行有效，如果在生产模式 运行，则断言不会执行


> 这篇文章的简单介绍就到这里了, 下一篇将会记录`Dart`的基本数据类型


### 参考文献

- [Dart官网语法介绍-中文版](http://dart.goodev.org/guides/language/language-tour)
- [Dart官网语法介绍-英文版](https://www.dartlang.org/guides/language/language-tour)

---



