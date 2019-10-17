
![NSRunLoop](https://titanjun.oss-cn-hangzhou.aliyuncs.com/ios/runloopmode.png)



<!--more-->


- 正常情况下, 一个线程执行完, 程序就会立即退出, 比如一个命令行项目
- `NSRunLoop`是`iOS`中的消息处理机制,执行完某个事件后线程不会退出，而是进入休眠状态，当再次监测到需要出发事件时，线程激活，继续处理事件，处理完成后再次进入休眠
- 这种时间运行循环, 类似于一个`while`循环
- 默认情况下, 不需要我们手动创建`RunLoop`, 因为`cocoa`框架为我们创建了一个默认的`RunLoop`
- `RunLoop`的主要作用
    - 保持程序的持续运行
    - 处理`App`中的各种事件（手势、定时器、`Selector`等）
    - 节省`CPU`资源、提高程序性能：该做任务的时候做任务，没事干的时候休息
- `RunLoop`和线程的关系
    - 每条线程都有唯一的一个与之对应的`RunLoop`对象
    - `RunLoop`保存在一个全局的`Dictionary`里, 线程作为`key`, `RunLoop`作为`value`
    - 线程刚创建时并没有`RunLoop`对象, `RunLoop`会在第一次获取线程时创建
    - `RunLoop`会在线程结束时销毁
    - 主线程的`RunLoop`已经自动获取(创建), 子线程默认没有开启`RunLoop`




## RunLoop对象

- 在`iOS`开发中`RunLoop`有两套`API`框架, 分别是
  - `Foundation`的`NSRunLoop`
  - `Core Foundation`的`CFRunLoopRef`
- `CFRunLoopRef`是基于`C`语言的开源框架, 有兴趣的可以到[源码地址](https://opensource.apple.com/tarballs/CF/)下载源码, 不过没有`C`语言功底的只怕很难看懂
- `NSRunLoop`是对`CFRunLoopRef`的有一层封装, 是`OC`语法的框架
- 简单使用, 获取`RunLoop`对象


```objc
// 获取当前线程的RunLoop
[NSRunLoop currentRunLoop];
// 获取主线程的RunLoop
[NSRunLoop mainRunLoop];
    
// 获取主线程的RunLoop
CFRunLoopGetMain();
// 获取当前线程的RunLoop
CFRunLoopGetCurrent();
```

### RunLoop相关的类

- 因为`NSRunLoop`是不开源的, 但是`CFRunLoopRef`却是开源的, 从[源码地址](https://opensource.apple.com/tarballs/CF/)下载`CFRunLoopRef`的源码
- 在源码中可以看到, 在`Core Foundation`中`CFRunLoopRef`有以下5个相关的类
    - `CFRunLoopRef`
    - `CFRunLoopModeRef`
    - `CFRunLoopSourceRef`
    - `CFRunLoopTimerRef`
    - `CFRunLoopObserverRef`


### `CFRunLoopRef`


`CFRunLoopRef`对象的主要核心代码如下

```objc
typedef struct __CFRunLoop * CFRunLoopRef;

struct __CFRunLoop {
    // 线程对象
    pthread_t _pthread;
    // 无序集合
    CFMutableSetRef _commonModes;
    CFMutableSetRef _commonModeItems;
    // 当前mode
    CFRunLoopModeRef _currentMode;
    CFMutableSetRef _modes;
};
```


<div class="note primary"><p>主要属性介绍</p></div>


- `CFMutableSetRef`是一个无序的集合, 在上面的代码中存储的都是`CFRunLoopModeRef`对象
- 其中`_modes`存储的是所有的`mode`对象
- `_currentMode`是指当前的`mode`


### `CFRunLoopModeRef`


```objc
typedef struct __CFRunLoopMode *CFRunLoopModeRef;

struct __CFRunLoopMode {
    // mode名称
    CFStringRef _name;
    Boolean _stopped;
    CFMutableSetRef _sources0;
    CFMutableSetRef _sources1;
    CFMutableArrayRef _observers;
    CFMutableArrayRef _timers;
};
```

- `_name`: 该`__CFRunLoopMode`的名称
- `_sources0`和`_sources1`: 一个无序集合, 存储的都是`CFRunLoopSourceRef`对象
- `_observers`: 一个有序集合数组,存储的都是`CFRunLoopObserverRef`对象
- `_timers`: 一个有序集合数组,存储的都是`CFRunLoopTimerRef`对象
- 从这里我们可以看出以上几个类之间的关系, 大概可如下图所示

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/ios/runloop_modes.png)

- `CFRunLoopModeRef`代表`RunLoop`的运行模式
- 一个`RunLoop`只能对应一个线程, 却包含若干个`Mode`，每个`Mode`又包含若干个`Source0/Source1/Timer/Observer`
- `RunLoop`启动时只能选择其中一个`Mode`，作为`currentMode`同样只能有一个
- 如果需要切换`Mode`，只能退出当前`Loop`，再重新选择一个`Mode`进入,
不同组的`Source0/Source1/Timer/Observer`能分隔开来，互不影响
- 如果`Mode`里没有任何`Source0/Source1/Timer/Observer`，`RunLoop`会立马退出
- 以下是系统默认的集中`mode`


```objc
FOUNDATION_EXPORT NSRunLoopMode const NSDefaultRunLoopMode;
FOUNDATION_EXPORT NSRunLoopMode const NSRunLoopCommonModes;

UIKIT_EXTERN NSRunLoopMode const UITrackingRunLoopMode;

CF_EXPORT const CFRunLoopMode kCFRunLoopDefaultMode;
CF_EXPORT const CFRunLoopMode kCFRunLoopCommonModes;
```

- `kCFRunLoopDefaultMode`（`NSDefaultRunLoopMode`）：`App`的默认`Mode`，通常主线程是在这个`Mode`下运行
- `UITrackingRunLoopMode`：界面跟踪`Mode`，用于`ScrollView`追踪触摸滑动，保证界面滑动时不受其他`Mode`影响
- `kCFRunLoopCommonModes`（`NSRunLoopCommonModes`）: 并不是某一种特定的`mode`, 而是通用模式, 包括`kCFRunLoopDefaultMode`和`UITrackingRunLoopMode`


### CFRunLoopObserverRef

`CFRunLoopObserverRef`是观察者，能够监听`RunLoop`所有的状态改变。
可以监听的时间点有如下几种：

```objc
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    // 即将进入Loop
    kCFRunLoopEntry = (1UL << 0),
    // 即将处理Timer
    kCFRunLoopBeforeTimers = (1UL << 1),
    // 即将处理Source
    kCFRunLoopBeforeSources = (1UL << 2),
    // 即将进入休眠
    kCFRunLoopBeforeWaiting = (1UL << 5),
    // 刚从休眠中唤醒
    kCFRunLoopAfterWaiting = (1UL << 6),
    // 即将退出Loop
    kCFRunLoopExit = (1UL << 7),
    // 所有状态
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```

在主线程监听所有的状态

```objc
    // 创建observer
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(kCFAllocatorDefault, kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
        switch (activity) {
            case kCFRunLoopEntry:
            NSLog(@"kCFRunLoopEntry");
            break;
            case kCFRunLoopBeforeTimers:
            NSLog(@"kCFRunLoopBeforeTimers");
            break;
            case kCFRunLoopBeforeSources:
            NSLog(@"kCFRunLoopBeforeSources");
            break;
            case kCFRunLoopBeforeWaiting:
            NSLog(@"kCFRunLoopBeforeWaiting");
            break;
            case kCFRunLoopAfterWaiting:
            NSLog(@"kCFRunLoopAfterWaiting");
            break;
            case kCFRunLoopExit:
            NSLog(@"kCFRunLoopExit");
            break;
            default:
            break;
        }
    });
    // 吧observer添加到RunLoop中
    CFRunLoopAddObserver(CFRunLoopGetMain(), observer, kCFRunLoopCommonModes);
    // 释放
    CFRelease(observer);
```

## RunLoop消息类型

![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/ios/runloop_date.png)

从上图可以看出消息类型大概可以分出两种, 第一种类型又可以细分为三种, 这三种都是异步执行的


### Port

监听程序的`mach ports`，`ports`可以简单的理解为：内核通过`port`这种方式将信息发送，而`mach`则监听内核发来的`port`信息，然后将其整理，打包发给`runloop`


### Customer

由开发人员自己发送, 苹果也提供了一个`CFRunLoopSource`来帮助处理, 简单介绍核心实:

1. 定义输入源（数据结构）
2. 将输入源添加到runloop，那么这样就有了接受者，即为R1
3. 协调输入源的客户端（单独线程），专门监听消息，然后将消息打包成`runloop`能够处理的样式，即第一步定义的输入源。它类似`Mach`的功能

### Selector Sources

NSObject类提供了很多方法供我们使用，这些方法是添加到runloop的，所以如果没有开启runloop的话，不会运行


```objc
/// 主线程
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
	
/// 指定线程
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait;

/// 针对当前线程, 延迟调用
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray<NSRunLoopMode> *)modes;
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay;

/// 取消，在当前线程，和上面两个方法对应
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(nullable id)anArgument;
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget;
```

- 上面提到的前四个方法是在指定线程运行`aSelector`, 一般情况下`aSelector`会添加到指定线程的`runloop`
- 如果调用线程和指定线程为同一线程，且`wait`参数设为`YES`，那么`aSelector`会直接在指定线程运行，不再添加到`runloop`; 
- 因为`wait`参数设为`YES`, 意味着要等待`aSelector`执行完成之后才回去执行后面的逻辑


## RunLoop运行逻辑

根据苹果在[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW23)里的说明，`RunLoop` 内部的逻辑大致如下:


![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/ios/runloopflow.png)


未查看`RunLoop`的执行流程, 我们可以新建一个项目, 并简单加一个触发事件, 如下所示


![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/ios/runloop_cycle.png)


- 如图所示, 添加一个简单的触发事件, 并加上断点, 在打印区域输入`bt`命令后, 就能看到完整的执行流程了
- 从下往上查看, 所执行的相关函数大概流程是:
- `UIApplicationMain`
- `CFRunLoopRunSpecific`
- `__CFRunLoopRun`
- `__CFRunLoopDoSources0`
- 最后就是`[UIResponder touchesBegan:withEvent:]`触发函数了
- 下面的事情就是找到源码, 依次查看所执行的函数了
- 在源码中找到`CFRunLoop.c`文件, 搜索`CFRunLoopRunSpecific`方法, 就是核心代码了, 一起来看看吧
- 删除其他不相关的代码, 核心代码大概如下


```objc
/// RunLoop的实现, 大概在文件的2622行
SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) { {
    
    /// 首先根据modeName找到对应mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(rl, modeName, false);
    
    /// 1. 通知 Observers: RunLoop 即将进入 loop。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopEntry);
    
    /// __CFRunLoopRun中具体要做的事情
    result = __CFRunLoopRun(rl, currentMode, seconds, returnAfterSourceHandled, previousMode);
    
    /// 11. 通知 Observers: RunLoop 即将退出。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
    
    return result;
}


// __CFRunLoopRun的实现, 进入loop, 大概在文件的2304行
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode) {

    int32_t retVal = 0;
    do {

        // 2. 通知 Observers: RunLoop 即将触发 Timer 回调。
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);
        
        // 3. 通知 Observers: RunLoop 即将触发 Source0 (非port) 回调。
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources);

        // 4. 处理block
        __CFRunLoopDoBlocks(rl, rlm);

        // 5. 处理Source0
        Boolean sourceHandledThisLoop = __CFRunLoopDoSources0(rl, rlm, stopAfterHandle);
        // 如果处理Source0的结果是rrue
        if (sourceHandledThisLoop) {
            // 再次处理block
            __CFRunLoopDoBlocks(rl, rlm);
        }

        Boolean poll = sourceHandledThisLoop || (0ULL == timeout_context->termTSR);

        // 6. 如果有Source1 (基于port) 处于ready状态，直接处理这个Source1然后跳转去处理消息。
        if (__CFRunLoopWaitForMultipleObjects(NULL, &dispatchPort, 0, 0, &livePort, NULL)) {
            // 如果有Source1, 就跳转到handle_msg
            goto handle_msg;
        }

        // 7. 通知 Observers: RunLoop 的线程即将进入休眠(sleep)
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeWaiting);
        __CFRunLoopSetSleeping(rl);
	
        // 7. 调用mach_msg等待接受mach_port的消息。线程将进入休眠, 等待别的消息来唤醒当前线程
        // 一个基于 port 的Source 的事件。
        // 一个 Timer 到时间了
        // RunLoop 自身的超时时间到了
        // 被其他什么调用者手动唤醒
        __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY, &voucherState, &voucherCopy);
        
        
        __CFRunLoopUnsetSleeping(rl);
        // 8. 通知Observers: 结束休眠, RunLoop的线程刚刚被唤醒了
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopAfterWaiting);

        
    // 收到消息，处理消息。
    handle_msg:;
        if (/* 被timer唤醒 */) {
            // 01. 处理Timer
            __CFRunLoopDoTimers(rl, rlm, mach_absolute_time())
        } else if (/* 被gcd唤醒 */) {
            // 02. 处理gcd
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
        } else {  // 被Source1唤醒
            // 处理Source1
            sourceHandledThisLoop = __CFRunLoopDoSource1(rl, rlm, rls) || sourceHandledThisLoop;
	    }
        
        // 9. 处理Blocks
        __CFRunLoopDoBlocks(rl, rlm);
        
        // 10. 设置返回值, 根据不同的结果, 处理不同操作
        if (sourceHandledThisLoop && stopAfterHandle) {
            // 进入loop时参数说处理完事件就返回。
            retVal = kCFRunLoopRunHandledSource;
        } else if (timeout_context->termTSR < mach_absolute_time()) {
            // 超出传入参数标记的超时时间了
            retVal = kCFRunLoopRunTimedOut;
        } else if (__CFRunLoopIsStopped(rl)) {
             // 被外部调用者强制停止了
            retVal = kCFRunLoopRunStopped;
        } else if (__CFRunLoopModeIsEmpty(rl, rlm, previousMode)) {
            // source/timer/observer一个都没有了
            retVal = kCFRunLoopRunFinished;
        }

        // 如果没超时，mode里没空，loop也没被停止，那继续loop。
    } while (0 == retVal);

    return retVal;
}
```

<div class="note success"><p>`RunLoop`</p></div>

从上面的代码可以看到`RunLoop`其内部是一个`do-while`循环; 当你调用`CFRunLoopRun()`时，线程就会一直停留在这个循环里；直到超时或被手动停止，该函数才会返回


## RunLoop的底层实现

- 从上面代码可以看到，`RunLoop`的核心是基于`mach port`的，其进入休眠时调用的函数是`mach_msg()`
- `Mach`本身提供的`API`非常有限，而且苹果也不鼓励使用`Mach`的`API`
- 但是这些`API`非常基础，如果没有这些`API`的话，其他任何工作都无法实施
- 在`Mach`中，所有的东西都是通过自己的对象实现的，进程、线程和虚拟内存都被称为”对象”
- 和其他架构不同，`Mach`的对象间不能直接调用，只能通过消息传递的方式实现对象间的通信。
- "消息"是`Mach`中最基础的概念，消息在两个端口 (`port`) 之间传递，这就是`Mach`的`IPC` (进程间通信) 的核心。


`Mach`的消息定义是在`<mach/message.h>`头文件的，很简单：

```objc
typedef struct {
  mach_msg_header_t header;
  mach_msg_body_t body;
} mach_msg_base_t;

typedef struct {
  mach_msg_bits_t msgh_bits;
  mach_msg_size_t msgh_size;
  mach_port_t msgh_remote_port;
  mach_port_t msgh_local_port;
  mach_port_name_t msgh_voucher_port;
  mach_msg_id_t msgh_id;
} mach_msg_header_t;
```

- 一条`Mach`消息实际上就是一个二进制数据包 (BLOB)，其头部定义了当前端口`local_port`和目标端口`remote_port`
- 发送和接受消息是通过同一个`API`进行的，其`option`标记了消息传递的方向：


```objc
mach_msg_return_t mach_msg(
	mach_msg_header_t *msg,
	mach_msg_option_t option,
	mach_msg_size_t send_size,
	mach_msg_size_t rcv_size,
	mach_port_name_t rcv_name,
	mach_msg_timeout_t timeout,
	mach_port_name_t notify
);
```


- 为了实现消息的发送和接收，`mach_msg()`函数实际上是调用了一个`Mach`陷阱(`trap`)，即函数`mach_msg_trap()`，陷阱这个概念在`Mach`中等同于系统调用
- 当你在用户态调用`mach_msg_trap()`时会触发陷阱机制，切换到内核态；内核态中内核实现的`mach_msg()`函数会完成实际的工作
- 内核态中的`mach_msg()`, 如果没有消息就让线程休眠,有消息就唤醒线程


![image](https://titanjun.oss-cn-hangzhou.aliyuncs.com/ios/runloop_trcp.png)


- `RunLoop`的核心就是一个`mach_msg()` (见上面代码的第7步)，`RunLoop`调用这个函数去接收消息，如果没有别人发送`port`消息过来，内核会将线程置于等待状态
- 例如你在模拟器里跑起一个`iOS`的`App`，然后在`App`静止时点击暂停，你会看到主线程调用栈是停留在`mach_msg_trap()`这个地方



## `NSRunLoop`应用实践

### NSTimer问题

解决`NSTimer`在滑动时停止工作的问题

- 上文有说到`CFRunLoopMode`主要使用的一般有三种`Mode`
- `DefaultMode`是`App`平时所处的状态，`TrackingRunLoopMode`是追踪`ScrollView`滑动时的状态
- 当你创建一个`Timer`并加到`DefaultMode`时，`Timer`会得到重复回调，但此时滑动一个`TableView`时，`RunLoop`会将`mode`切换为`TrackingRunLoopMode`，这时`Timer`就不会被回调，并且也不会影响到滑动操作
- 下面我来看一下这个例子


```objc
#import "ViewController.h"

@interface ViewController ()
// 在xib中添加一个可滚动的UITextView
@property (weak, nonatomic) IBOutlet UITextView *textView;
@property (assign, nonatomic) NSInteger timerCount;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 添加一个定时器
    [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(timerClick) userInfo:nil repeats:YES];
}
    
- (void)timerClick {
    NSLog(@"--------%ld", (long)self.timerCount++);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSLog(@"touchesBegan");
}
@end
```

- 上面代码中`scheduledTimerWithTimeInterval`方式添加的`NSTimer`会默认被添加到`DefaultMode`中
- 当程序运行的时候回正常执行定时器的方法
- 当我们正常滚动`UITextView`的时候, 从打印结果可以看到, 定时器停止执行了, 结束滚动`UITextView`的时候, 定时器方法会继续执行
- 输出结果如下所示


```
2019-08-20 21:46:01.986843+0800 RunLoop[86811:3484205] --------0
2019-08-20 21:46:02.986723+0800 RunLoop[86811:3484205] --------1
2019-08-20 21:46:03.986040+0800 RunLoop[86811:3484205] --------2
2019-08-20 21:46:04.986274+0800 RunLoop[86811:3484205] --------3
2019-08-20 21:46:05.272525+0800 RunLoop[86811:3484205] touchesBegan
2019-08-20 21:46:12.291035+0800 RunLoop[86811:3484205] --------4
2019-08-20 21:46:12.986318+0800 RunLoop[86811:3484205] --------5
2019-08-20 21:46:13.986197+0800 RunLoop[86811:3484205] --------6
2019-08-20 21:46:14.986735+0800 RunLoop[86811:3484205] --------7
```

- 有时你需要一个`Timer`，在两个`Mode`中都能得到回调
- 一种办法就是将这个`Timer`分别加入这两个`Mode`
- 另一种方式，就是将`Timer`加入到顶层的`RunLoop`的`commonModeItems`中
- `commonModeItems`被`RunLoop`自动更新到所有具有`Common`属性的`Mode`里去
- `CommonModes`并不是一个真的模式，它只是一个标记


```objc
NSTimer *timer = [NSTimer timerWithTimeInterval:1 repeats:YES block:^(NSTimer * _Nonnull timer) {
    NSLog(@"--------%ld", (long)self.timerCount++);
}];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```


---
