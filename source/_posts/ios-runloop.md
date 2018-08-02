---
title: iOS Runloop 深入浅出
date: 2018-07-28 14:38:09
tags: iOS
categories: iOS
---

# iOS Runloop 深入浅出

## Runloop是什么？

苹果官方文档解释：

> A run loop is a piece of infrastructure used to manage events arriving asynchronously on a thread. A run loop works by monitoring one or more event sources for the thread. As events arrive, the system wakes up the thread and dispatches the events to the run loop, which then dispatches them to the handlers you specify. If no events are present and ready to be handled, the run loop puts the thread to sleep.

根据以上这个段落，我们能够得知一个重要的信息，**Runloop**是用来处理异步达到**线程**上事件的一种基础结构。所以，要想理解好**Runloop**，我们首先得把**线程**理解清楚了，许多同学跳过线程直接去理解**Runloop**(更不可取的是有部分同学只是为了应付面试而死背硬记Runloop相关知识，往往过后就忘记了)，往往搞得一头雾水，不理解为什么要有这种东西，即使网络上看了几篇直接讲述**Runloop**并贴有很多源代码分析的文章，也始终没有搞懂来龙去脉。要记住，**Runloop**仅仅只是管理事件异步达到线程的一种结构而已，其目的是解决多线程管理中的问题（线程管理中，还有其他手段可以实现类似Runloop这种机制，例如使用信号量, 只不过依赖Runloop更为简单）。

## 线程

偷个懒，还是借用苹果官方文档的解释来解释什么是线程:]

> Each process (application) in OS X or iOS is made up of one or more threads, each of which represents a single path of execution through the application's code. Every application starts with a single thread, which runs the application's main function. Applications can spawn additional threads, each of which executes the code of a specific function.
> 
> When an application spawns a new thread, that thread becomes an independent entity inside of the application's process space. Each thread has its own execution stack and is scheduled for runtime separately by the kernel. A thread can communicate with other threads and other processes, perform I/O operations, and do anything else you might need it to do. Because they are inside the same process space, however, all threads in a single application share the same virtual memory space and have the same access rights as the process itself.

大意就是每个OS X或者iOS每个应用程序都会以一个线程开始，这个线程执行应用程序的main方法。应用程序可以产生多个线程，每个线程在应用程序内都会有独立的处理空间、堆栈并且被kernel分别在运行时安排执行时间。一个线程可以与同进程内的其他线程通信甚至其他进程、执行I/O操作或者是任何你想让它做的事情。

### 为什么需要有线程？

在传统的操作系统中，进程拥有独立的内存地址空间和一个用于控制的线程。但是，现在的情况更多的情况下要求在同一地址空间下拥有多个线程并发执行。因此线程被引入操作系统。

 如果非要说是为什么需要线程，还不如说为什么需要进程中还有其它进程。这些进程中包含的其它迷你进程就是线程线程之所以说是迷你进程，是因为线程和进程有很多相似之处，比如线程和进程的状态都有运行，就绪，阻塞状态。这几种状态理解起来非常简单，当进程所需的资源没有到位时会是阻塞状态，当进程所需的资源到位时但CPU没有到位时是就绪状态，当进程既有所需的资源，又有CPU时，就为运行状态。

假设没有线程，相当于进程只有一个线程，很多情况下的用户体验是无法接受的。来看一个具体的例子，使用iPhone拍摄视频或者照片后，相机app需要将照片或者视频写入磁盘，而访问磁盘的时间内线程是阻塞的，这个时候在相机app无论执行什么操作都不会相应，因为仅有的线程在忙着处理数据写入呢。而如果引入了多线程，每个线程仅仅需要处理自己那一部分应该完成的任务，而不用去关心和其他线程的冲突，因此简化了编程模型。

### 创建一个线程

使用**NSThread**创建线程

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(onThreadStart:) object:nil];
    self.thread.name = @"myThread";
    [self.thread start];
}

- (void)onThreadStart:(id)args {
    NSLog(@"Execution in thread [%@]", [NSThread currentThread].name);
}

- (void)persistVideoClips {
    //这里有存储视频信息的代码
}

```

可以看到控制台打印`Execution in thread [myThread]`

### 使用线程

继续前面讲的存储视频的例子，我们在界面加上一个按钮，每次点击就使用线程模拟保存视频(也许你会说，可以使用GCD或者NSOperationQueue来完成啊！对，但是这两个也都是苹果提供的多线程管理库，我们要研究为什么苹果引入Runloop就需要从线程上去考虑)。

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    self.videoClips = [NSMutableArray array];
}

- (IBAction)onButtonTapped:(id)sender {
   self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(onThreadStart:) object:[self.videoClips copy]];
    self.thread.name = @"myThread";
    [self.thread start];
}

- (void)onThreadStart:(id)args {
    //这里有存储视频信息的代码
    @autoreleasepool {
		NSLog(@"Execution in thread [%@]", [NSThread currentThread].name);
		NSLog(@"saving video clips");
    }
}
```

这种方式是强烈不推荐的，想必也没有人会这么干，因为线程的创建是昂贵的。

> Threading has a real cost to your program (and the system) in terms of memory use and performance. Each thread requires the allocation of memory in both the kernel memory space and your program’s memory space. The core structures needed to manage your thread and coordinate its scheduling are stored in the kernel using wired memory. Your thread’s stack space and per-thread data is stored in your program’s memory space. Most of these structures are created and initialized when you first create the thread—a process that can be relatively expensive because of the required interactions with the kernel.

为了解决消耗这个问题，聪明的人对线程进行了封装，使用继承实现了自己的`LCThread`

```objc
/**
 * LCThread.h
 */
 
typedef void (^LCActionBlock)(void);

@interface LCThread : NSThread
- (void)queueAction:(LCActionBlock)action;
@end

/**
 * LCThread.m
 */

#import "LCThread.h"

@interface LCThread ()
@property (strong, nonatomic) NSMutableArray<LCActionBlock> *actions;
@property (strong, nonatomic) NSObject *mutex;
@end

@implementation LCThread

- (instancetype)init
{
    self = [super init];
    if (self) {
        _actions = [NSMutableArray array];
        _mutex   = [NSObject new];
    }
    return self;
}

- (void)queueAction:(LCActionBlock)action {
    @synchronized(self.mutex) {
        [self.actions addObject:[action copy]];
    }
}

- (void)main {
    while (true) {
    	  @autoreleasepool {
		    @synchronized(self.mutex) {
		        NSArray<LCActionBlock> *actions = [self.actions copy];
		        [self.actions removeAllObjects];
		        
		        [actions enumerateObjectsUsingBlock:^(LCActionBlock  _Nonnull action, NSUInteger idx, BOOL * _Nonnull stop) {
		            @try {
		                action();
		            } @catch(NSException *error) {
		                NSLog(@"error occurred: %@", error);
		            }
		        }];
		    }
        }
        [NSThread sleepForTimeInterval:0.3];
    }
}

@end

/*
 * 主界面代码
 */

- (void)viewDidLoad {
    [super viewDidLoad];
    self.thread = [LCThread new];
    self.thread.name = @"myThread";
    [self.thread start];
}

- (IBAction)onButtonTapped:(id)sender {
    [self.thread queueAction:^{
        NSLog(@"Execution in thread [%@]", [NSThread currentThread].name);
        NSLog(@"saving video clips");
    }];
}

```

现在点击按钮，可以看到`video clips`正确地在我们创建的线程中调用，基本达到我们让保存视频的代码在后台线程执行的目的。但是这种方法有缺点：

- 为了让线程一直运行但又不占满CPU时间，我们引入了`[NSThread sleepForTimeInterval:0.3]`，人为地让线程“活着”
- 即使`actions`为空，线程也需要从休眠中醒来检查还有没有需要处理的`action`，带来了无意义的消耗
- 无法随心所欲地唤醒线程，只能被动等待线程休眠足够长的时间
- 无法对`LCThread`使用`performSelector:onThread:withObject:waitUntilDone`等方法

为了解决包括这个例子遇到的问题，苹果引入了`Runloop`的实现。

## Runloop

现在再来拜读一下苹果对`Runloop`的解释

>Run loops are part of the fundamental infrastructure associated with threads. A run loop is an event processing loop that you use to schedule work and coordinate the receipt of incoming events. The purpose of a run loop is to keep your thread busy when there is work to do and put your thread to sleep when there is none.

这不正是我们想要的么？当有工作需要线程做的时候，就让它干活，不需要它的时候，就让它休息去。“召之即来挥之即去”，在休眠时几乎不占用系统资源。

### 尝试使用Runloop

苹果不允许直接创建`Runloop`，它只提供了两个自动获取的函数：`CFRunloopGetMain()`和`CFRunloopGetCurrent()`。

```c
/// 全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
static CFMutableDictionaryRef loopsDic;
/// 访问 loopsDic 时的锁
static CFSpinLock_t loopsLock;
 
/// 获取一个 pthread 对应的 RunLoop。
CFRunLoopRef _CFRunLoopGet(pthread_t thread) {
    OSSpinLockLock(&loopsLock);
    
    if (!loopsDic) {
        // 第一次进入时，初始化全局Dic，并先为主线程创建一个 RunLoop。
        loopsDic = CFDictionaryCreateMutable();
        CFRunLoopRef mainLoop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, pthread_main_thread_np(), mainLoop);
    }
    
    /// 直接从 Dictionary 里获取。
    CFRunLoopRef loop = CFDictionaryGetValue(loopsDic, thread));
    
    if (!loop) {
        /// 取不到时，创建一个
        loop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, thread, loop);
        /// 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
        _CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
    }
    
    OSSpinLockUnLock(&loopsLock);
    return loop;
}
 
CFRunLoopRef CFRunLoopGetMain() {
    return _CFRunLoopGet(pthread_main_thread_np());
}
 
CFRunLoopRef CFRunLoopGetCurrent() {
    return _CFRunLoopGet(pthread_self());
}
```

现在就让我们在创建的线程中使用它吧

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    self.videoClips = [NSMutableArray array];
    self.thread = [[NSThread alloc] initWithBlock:^{
        NSLog(@"Execution in thread [%@]", [NSThread currentThread].name);
        [[NSRunLoop currentRunLoop] run];
    }];
    self.thread.name = @"myThread";
    [self.thread start];
}

- (IBAction)onButtonTapped:(id)sender {
    [self performSelector:@selector(persistVideoClips:) onThread:self.thread withObject:[self.videoClips copy] waitUntilDone:NO];
}

- (void)persistVideoClips:(NSArray *)clips {
    NSLog(@"saving video clips");
}
```

啊啊啊~！按照我们预想的结果，运行以后每次点击屏幕都应该有输出，但是实际上我们点击屏幕并没有任何效果，没有打印`saving video clips`。

根据[苹果官方文档 - Starting the Run loop](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW25)的解释

> A run loop must have at least one input source or timer to monitor. If one is not attached, the run loop exits immediately.

因为没有`input source`或者`timer source`，所以`Runloop`在启动后就立即退出了。

### 深入Runloop

不对Runloop一番了解，很难驾驭，所以我们就先来熟悉熟悉Runloop。Runloop在源代码中体现为一个对象，指的是**NSRunloop**或者**CFRunloopRef**，CFRunloopRef是在 CoreFoundation 框架内的，它提供了纯C的函数API，所有这些 API 都是线程安全的，而NSRunloop仅仅是CFRunloopRef的objective-c的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。CFRunLoopRef 的代码是开源的，你可以在这里[http://opensource.apple.com/tarballs/CF/](http://opensource.apple.com/tarballs/CF/)下载到整个 CoreFoundation 的源码来查看。我们接下来通过代码来了解它的内部逻辑(为了阅读方便，提取了重要的部分展示，如果不想阅读可以直接跳过看后面的流程图)：

```c
//使用kCFRunLoopDefaultMode启动Runloop
void CFRunLoopRun(void) {	/* DOES CALLOUT */
    int32_t result;
    do {
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}

//使用指定的mode启动Runloop
SInt32 CFRunLoopRunInMode(CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {     /* DOES CALLOUT */
    CHECK_FOR_FORK();
    return CFRunLoopRunSpecific(CFRunLoopGetCurrent(), modeName, seconds, returnAfterSourceHandled);
}


SInt32 CFRunLoopRunSpecific(runloop, modeName, seconds, returnAfterSourceHandled) {     /* DOES CALLOUT */
    if (__CFRunLoopIsDeallocating(runloop)) 
        return kCFRunLoopRunFinished;

    //首先根据modeName找到对应的mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(runloop, modeName, false);

    //如果mode为空，或者mode里面没有[sources,timers,observers]直接返回
    if (NULL == currentMode || __CFRunLoopModeIsEmpty(runloop, currentMode, runloop->_currentMode)) {
    	return kCFRunLoopRunFinished;
    }

    int32_t result = kCFRunLoopRunFinished;

    //通知observers，即将进入loop
	if (currentMode->_observerMask & kCFRunLoopEntry ) 
        __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopEntry);

	//调用内部函数进入loop
	result = __CFRunLoopRun(runloop, currentMode, seconds, returnAfterSourceHandled, previousMode);

    //通知observers，Runloop即将退出
	if (currentMode->_observerMask & kCFRunLoopExit )
        __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopExit);

    return result;
}

//进入loop的内部函数
static int32_t __CFRunLoopRun(runloop, currentMode, seconds, stopAfterHandle) {
    int32_t retVal = 0;
    Boolean isTimerSource = false;
    //获得timer的port
    mach_port_name_t timerPort = MACH_PORT_NULL;
    if (runloop->_queue) {
        timerPort = _dispatch_runloop_root_queue_get_port_4CF(runloop->_queue);
        if (!timerPort) {
            CRASH("Unable to get port for run loop mode queue (%d)", -1);
        }
    }

    do {
        // 2. 通知 Observers: RunLoop 即将触发 Timer 回调。
        __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeTimers);
        // 3. 通知 Observers: RunLoop 即将触发 Source0 (非port) 回调。
        __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeSources);
        // 执行被加入的block
        __CFRunLoopDoBlocks(runloop, currentMode);
        // 4. RunLoop 触发 Source0 (非port) 回调。
        Boolean sourceHandledThisLoop = __CFRunLoopDoSources0(runloop, currentMode, stopAfterHandle);
        if (sourceHandledThisLoop) {
            __CFRunLoopDoBlocks(runloop, currentMode);
        }

        //5. 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去9
        if (MACH_PORT_NULL != dispatchPort && !didDispatchPortLastTime) {
            msg = (mach_msg_header_t *)msg_buffer;
            if (__CFRunLoopServiceMachPort(dispatchPort, &msg, sizeof(msg_buffer), &livePort, 0, &voucherState, NULL)) {
                goto handle_msg;
            }
        }

        // 通知 Observers: RunLoop的线程即将进入休眠
        if (!sourceHandledThisLoop) {
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
        }

        // 7. 调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
        // • 一个基于 port 的Source 的事件。
        // • 一个 Timer 到时间了
        // • RunLoop 自身的超时时间到了
        // • 被其他什么调用者手动唤醒
        __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY, &voucherState, &voucherCopy);

        isTimerSource = timerPort != MACH_PORT_NULL && livePort == timerPort;
        // 8. 通知 Observers: Runloop的线程刚刚被唤醒了
        __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopAfterWaiting);

    handle_msg:
        // 9.1 如果是timer唤醒，触发这个Timer的回调。
        if(isTimerSource) {
            __CFRunLoopDoTimers(runloop, currentMode, mach_absolute_time()
        //9.2 如果是main dispatch的唤醒，执行block
        } else if(msg_is_dispatch) {
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
        //9.3 如果一个 Source1 (基于port) 唤醒，处理source1
        } else {
            CFRunLoopSourceRef source1 = __CFRunLoopModeFindSourceForMachPort(runloop, currentMode, livePort);
            sourceHandledThisLoop = __CFRunLoopDoSource1(runloop, currentMode, source1, msg);
            if (sourceHandledThisLoop) {
                mach_msg(reply, MACH_SEND_MSG, reply);
            }
        }

        //  再次确保是否有同步的方法需要调用
        __CFRunLoopDoBlocks(runloop, currentMode);

        if (sourceHandledThisLoop && stopAfterHandle) {
        	// 进入loop时参数说处理完事件就返回。
	       retVal = kCFRunLoopRunHandledSource;
        } else if (timeout) {
           // 超出传入参数标记的超时时间了
           retVal = kCFRunLoopRunTimedOut;
        } else if (__CFRunLoopIsStopped(runloop)) {
           // 被外部调用者强制停止了
           retVal = kCFRunLoopRunStopped;
        } else if (currentMode->_stopped) {
        	// currentMode 被强制停止，调用内部_CFRunLoopStopMode
        	currentMode->_stopped = false;
        	retVal = kCFRunLoopRunStopped;
		 } else if (__CFRunLoopModeIsEmpty(runloop, currentMode)) {
		 	// source/timer/observer一个都没有了
		 	retVal = kCFRunLoopRunFinished;
        }
    } while(0 == retVal);
    return retVal;
}
```
通过上面的伪代码可以看到，实际上Runloop就是内部是一个do-while调用的函数。这个函数管理线程需要处理的事件和消息，并在没有处理消息时休眠以避免占用系统资源，在有消息到来时立刻被唤醒。线程执行了`Runloop`后，其内部就会一直处于`接收消息->休眠->处理消息`的循环中，直到循环结束条件满足才会退出，结束线程。

下图展示了Runloop运行时的关键流程：

![](http://ot51d7lis.bkt.clouddn.com/Runloop2.png)

通过代码和流程图我们有了初步的了解，但是还是有种神秘的感觉，别急，接下来就为您一一介绍什么是mode，source, observers和callouts。

#### Runloop modes

从源码很容易看出，Runloop总是运行在某种特定的CFRunLoopModeRef下，每次运行**CFRunLoopRun**函数时必须指定一种Mode，这个mode称为**current mode**。通过**CFRunloopRef**的结构体可以看出，一个Runloop可以包含N个modes，每个mode又可以包含若干个**Sources/Timers/Observers**。当切换Mode时必须退出**current mode**，然后再重新进入。

![](http://ot51d7lis.bkt.clouddn.com/RunloopModes.png) 

CFRunloopMode和CFRunloop的结构如下：

```c
struct __CFRunLoopMode {
	 CFRuntimeBase _base;
    pthread_mutex_t _lock;	/* must have the run loop locked before locking this */
    CFStringRef _name;
    Boolean _stopped;
    char _padding[3];
    CFMutableSetRef _sources0; // source0
    CFMutableSetRef _sources1; // source1
    CFMutableArrayRef _observers;
    CFMutableArrayRef _timers;
    CFMutableDictionaryRef _portToV1SourceMap;
    __CFPortSet _portSet;
    CFIndex _observerMask;
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    dispatch_source_t _timerSource;
    dispatch_queue_t _queue;
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
#if USE_MK_TIMER_TOO
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    uint64_t _timerSoftDeadline; /* TSR */
    uint64_t _timerHardDeadline; /* TSR */
};
 
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;			/* locked for accessing mode list */
    __CFPort _wakeUpPort;			// used for CFRunLoopWakeUp 
    Boolean _unused;
    volatile _per_run_data *_perRunData;              // reset for runs of the run loop
    pthread_t _pthread;
    uint32_t _winthread;
    CFMutableSetRef _commonModes;
    CFMutableSetRef _commonModeItems;
    CFRunLoopModeRef _currentMode;
    CFMutableSetRef _modes;
    struct _block_item *_blocks_head;
    struct _block_item *_blocks_tail;
    CFAbsoluteTime _runTime;
    CFAbsoluteTime _sleepTime;
    CFTypeRef _counterpart;
};
```

##### Common Modes

一个 Mode 可以将自己标记为`"Common"`属性（通过将其 ModeName 添加到RunLoop的 `_commonModes` 中）。每当 RunLoop的内容发生变化时，RunLoop都会自动将_commonModeItems里的 Source/Observer/Timer 同步到具有 `“Common”` 标记的所有Mode里。

###### 应用场景举例：

主线程的RunLoop里有两个预置的 Mode：`kCFRunLoopDefault	Mode` 和 `UITrackingRunLoopMode`。这两个 Mode 都已经被标记为”Common”属性。`DefaultMode` 是 App 平时所处的状态，`TrackingRunLoopMode` 是追踪 ScrollView 滑动时的状态。当你创建一个 Timer 并加到 DefaultMode时，Timer会得到重复回调，但此时滑动一个TableView时，RunLoop会将 mode 切换为 `TrackingRunLoopMode`，这时 Timer 就不会被回调，并且也不会影响到滑动操作。如果你想让这个Timer，在两个 Mode 中都能得到回调，有三种办法：

- 将这个`Timer`分别加入这两个 Mode
- 将`Timer`加入到顶层的 RunLoop 的 “commonModeItems” 中。”commonModeItems” 被 RunLoop 自动更新到所有具有”Common”属性的 Mode 里去
- 将`Timer`加入到kCFRunLoopCommonModes（后面小节会提到）里面

##### 常见modes介绍

- `NSDefaultRunLoopMode`：App的默认 Mode。这个mode应该用在线程默认、空闲和等待事件时。
- `UITrackingRunLoopMode`：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
- `UIInitializationRunLoopMode`： 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
- `GSEventReceiveRunLoopMode`： 接受系统事件的内部 Mode，通常用不到
- `NSRunLoopCommonModes`：通过将对象（timer/source/observer）通过`CFRunLoopAddCommonMode`加入到`Runloop`后，能够被同步到所有标记为`"common"`的modes里

##### 常用接口

```c
///将一个mode加入到common modes的set里面
CFRunLoopAddCommonMode(CFRunLoopRef runloop, CFStringRef modeName);
///以指定mode在当前线程启动runloop
CFRunLoopRunResult CFRunLoopRunInMode(CFRunLoopMode mode, CFTimeInterval seconds, Boolean returnAfterSourceHandled);

///添加一个source到指定的mode
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
///判断指定的mode是否包含传入的source
CFRunLoopContainsSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFRunLoopMode mode);
///将一个source从指定mode中移除
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);

///添加一个observer到指定的mode
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
///判断指定的mode是否包含传入的observer
CFRunLoopContainsObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFRunLoopMode mode);
///将一个observer从指定mode中移除
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);

///添加一个timer到指定的mode
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
///将timer从指定的mode中移除
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```

#### Sources

Sources是用来唤醒Runloop的线程的

##### CFRunLoopSourceRef

CFRunLoopSourceRef对应两种源: source0和source1。其中Source1和Timer source都属于端口事件源，不同的是所有的Timer都共用一个端口，而每个source1都有自己的端口。

- source0：负责app内部事件，只包含一个回调函数指针，不能主动触发事件。使用时，需要先调用**CFRunLoopSourceSignal(source)**,将这个Source标记为待处理，然后手动调用 **CFRunLoopWakeUp(runloop)** 来唤醒 RunLoop，让其处理这个事件
- Source1：除了包含回调指针外还包含一个**mach port**，和source0不同，source1可以监听系统端口或者和其他线程互发消息，它能够主动唤醒Runloop。

##### CFRunLoopTimerRef

Timer会在一个预设的时间内向线程传递**同步**消息。Timer是与Runloop的mode来联系在一起的，如果Timer所在的mode没有被Runloop所使用，那么它将不会被调用直到Runloop切换到Timer所属的mode。同样地，如果Timer在Runloop处理事务过程中触发了，那就需要等到Runloop下一次loop才能执行Timer的回调了。如果Runloop压根就没有跑起来，那么Timer无论如何也不会触发的。

#### Observers

与sources相反，Runloop observers是用来观察Runloop的状态变化的。当Runloop状态改变，可以通过observers的回调接收到相应的变化。

```c
struct __CFRunLoopObserver {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFIndex _rlCount;
    CFOptionFlags _activities;      /* immutable */
    CFIndex _order;         /* immutable */
    CFRunLoopObserverCallBack _callout; /* immutable */
    CFRunLoopObserverContext _context;  /* immutable, except invalidation */
};
```
可以观察到的状态有：

```c
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0), // 进入RunLoop 
    kCFRunLoopBeforeTimers = (1UL << 1), // 即将开始Timer处理
    kCFRunLoopBeforeSources = (1UL << 2), // 即将开始Source处理
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6), //从休眠状态唤醒
    kCFRunLoopExit = (1UL << 7), //退出RunLoop
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```

#### Call out

在开发过程中几乎所有的操作都是通过Call out进行回调的(无论是Observer的状态通知还是Timer、Source的处理)，而系统在回调时通常使用如下几个函数进行回调(换句话说你的代码其实最终都是通过下面几个函数来负责调用的，即使你自己监听Observer也会先调用下面的函数然后间接通知你，所以在调用堆栈中经常看到这些函数)：

```c
static void __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__() __attribute__((noinline));
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__() __attribute__((noinline));
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__() __attribute__((noinline));
static void __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__() __attribute__((noinline));
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__() __attribute__((noinline));
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__() __attribute__((noinline));
```
#### Runloop的休眠

RunLoop最核心的事情就是保证线程在没有消息时休眠以避免占用系统资源，有消息时能够及时唤醒。RunLoop的这个机制完全依靠系统内核来完成，具体来说是苹果操作系统核心组件Darwin中的Mach来完成的（[Darwin](https://opensource.apple.com/)是开源的）

```c
/*
 *  Routine:    mach_msg
 *  Purpose:
 *      Send and/or receive a message.  If the message operation
 *      is interrupted, and the user did not request an indication
 *      of that fact, then restart the appropriate parts of the
 *      operation silently (trap version does not restart).
 */
__WATCHOS_PROHIBITED __TVOS_PROHIBITED
extern mach_msg_return_t    mach_msg(
                    mach_msg_header_t *msg,
                    mach_msg_option_t option,
                    mach_msg_size_t send_size,
                    mach_msg_size_t rcv_size,
                    mach_port_name_t rcv_name,
                    mach_msg_timeout_t timeout,
                    mach_port_name_t notify);
```

### 使用Runloop

#### 使用NSMachPort作为source (source1)

经过上一节的分析，已经对Runloop有了深入的了解。现在就能修改之前的例子，让它能正常工作起来了。为了让Runloop不退出，最简单的方式就是创建一个NSMachPort添加到Runloop里面。

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    self.videoClips = [NSMutableArray array];
    self.thread = [[NSThread alloc] initWithBlock:^{
        NSRunLoop *runloop = [NSRunLoop currentRunLoop];
        [runloop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runloop run];
    }];
    self.thread.name = @"myThread";
    [self.thread start];
}

- (IBAction)onButtonTapped:(id)sender {
    [self performSelector:@selector(persistVideoClips:) onThread:self.thread withObject:[self.videoClips copy] waitUntilDone:NO];
}

- (void)persistVideoClips:(NSArray *)clips {
    NSLog(@"Execution in thread [%@], saving video clips", [NSThread currentThread].name);
}
```

运行后，点击按钮，可以看到控制台输出`Execution in thread [myThread], saving video clips`。

#### 自定义source (Source0)

我们把例子用source0的方式修改一下

```objc
void runloopSourceScheduleRoutine(void *info, CFRunLoopRef runloop, CFRunLoopMode mode) {
    NSLog(@"Schedule routine: source is added to runloop");
}

void runloopSourceCancelRoutine(void *info, CFRunLoopRef runloop, CFRunLoopMode mode) {
    NSLog(@"Cancel Routine: source removed from runloop");
}

void runloopSourcePerformRoutine(void *info) {
    NSLog(@"In thread [%@], Perform Routine: source has fired", [NSThread currentThread].name);
}

@interface MyThread ()
@property (assign, nonatomic) CFRunLoopSourceRef source;
@property (assign, nonatomic) CFRunLoopRef runloop;
@end

@implementation MyThread

- (void)dealloc {
    self.source = NULL;
    self.runloop = NULL;
}

- (void)main {
    @autoreleasepool {
        NSRunLoop *runloop = [NSRunLoop currentRunLoop];
        self.runloop = [runloop getCFRunLoop];
        CFRunLoopSourceContext context = {0, (__bridge void *)(self), NULL, NULL, NULL, NULL, NULL, runloopSourceScheduleRoutine, runloopSourceCancelRoutine, runloopSourcePerformRoutine };
        self.source = CFRunLoopSourceCreate(NULL, 0, &context);
        CFRunLoopAddSource(self.runloop, self.source, kCFRunLoopDefaultMode);
        [runloop run];
    }
}

- (void)triggerSource0 {
    CFRunLoopSourceSignal(self.source);
    CFRunLoopWakeUp(self.runloop);
}

/**
 * ViewController 
 */

- (void)viewDidLoad {
    [super viewDidLoad];
    self.videoClips = [NSMutableArray array];
    self.thread = [MyThread new];
    self.thread.name = @"myThread";
    [self.thread start];
}

- (IBAction)buttonTapped:(id)sender {
    [self.thread triggerSource0];
}

```

#### 使用Observer

```objc
void addObserver(CFRunLoopRef runloop) {
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(
                                                                       kCFAllocatorDefault,
                                                                       kCFRunLoopAllActivities,
                                                                       YES,
                                                                       0,
       ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
           /*
            kCFRunLoopEntry = (1UL << 0),          进入工作
            kCFRunLoopBeforeTimers = (1UL << 1),   即将处理Timers事件
            kCFRunLoopBeforeSources = (1UL << 2),  即将处理Source事件
            kCFRunLoopBeforeWaiting = (1UL << 5),  即将休眠
            kCFRunLoopAfterWaiting = (1UL << 6),   被唤醒
            kCFRunLoopExit = (1UL << 7),           退出RunLoop
            kCFRunLoopAllActivities = 0x0FFFFFFFU  监听所有事件
            */
           switch (activity) {
               case kCFRunLoopEntry:
                   NSLog(@"进入");
                   break;
               case kCFRunLoopBeforeTimers:
                   NSLog(@"即将处理Timer事件");
                   break;
               case kCFRunLoopBeforeSources:
                   NSLog(@"即将处理Source事件");
                   break;
               case kCFRunLoopBeforeWaiting:
                   NSLog(@"即将休眠");
                   break;
               case kCFRunLoopAfterWaiting:
                   NSLog(@"被唤醒");
                   break;
               case kCFRunLoopExit:
                   NSLog(@"退出RunLoop");
                   break;
               default:
                   break;
           }
       }
  );
    CFRunLoopAddObserver(runloop, observer, kCFRunLoopDefaultMode);
}

@interface ViewController ()
@property (weak ,nonatomic) NSTimer *timer;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.timer = [NSTimer scheduledTimerWithTimeInterval:2.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
        NSLog(@"timer tick");
    }];
    addObserver(CFRunLoopGetCurrent());
}

@end
```

控制台打印的部分信息：

> 2018-08-02 14:46:40.994863+0800 ObserverDemo[4707:559984] 被唤醒
2018-08-02 14:46:40.995346+0800 ObserverDemo[4707:559984] 即将处理Timer事件
2018-08-02 14:46:40.995468+0800 ObserverDemo[4707:559984] 即将处理Source事件
2018-08-02 14:46:40.995584+0800 ObserverDemo[4707:559984] 即将休眠
2018-08-02 14:46:41.584618+0800 ObserverDemo[4707:559984] 被唤醒
2018-08-02 14:46:41.584982+0800 ObserverDemo[4707:559984] timer tick
2018-08-02 14:46:41.585309+0800 ObserverDemo[4707:559984] 即将处理Timer事件
2018-08-02 14:46:41.585592+0800 ObserverDemo[4707:559984] 即将处理Source事件
2018-08-02 14:46:41.585810+0800 ObserverDemo[4707:559984] 即将休眠
2018-08-02 14:46:43.584045+0800 ObserverDemo[4707:559984] 被唤醒
2018-08-02 14:46:43.584247+0800 ObserverDemo[4707:559984] timer tick
2018-08-02 14:46:43.584424+0800 ObserverDemo[4707:559984] 即将处理Timer事件
2018-08-02 14:46:43.584539+0800 ObserverDemo[4707:559984] 即将处理Source事件
2018-08-02 14:46:43.584685+0800 ObserverDemo[4707:559984] 即将休眠
2018-08-02 14:46:45.584768+0800 ObserverDemo[4707:559984] 被唤醒
2018-08-02 14:46:45.585063+0800 ObserverDemo[4707:559984] timer tick
2018-08-02 14:46:45.585379+0800 ObserverDemo[4707:559984] 即将处理Timer事件
2018-08-02 14:46:45.585638+0800 ObserverDemo[4707:559984] 即将处理Source事件
2018-08-02 14:46:45.585864+0800 ObserverDemo[4707:559984] 即将休眠

可以看到流程跟我们在深入Runloop分析的一样。

### 参考资料

苹果的[Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i-CH1-SW1)

Kenshin的[iOS刨根问底-深入理解Runloop](https://www.cnblogs.com/kenshincui/p/6823841.html)

ibireme的[深入理解Runloop](https://blog.ibireme.com/2015/05/18/runloop/)