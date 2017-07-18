---
title: Memory leak 如果你使用AFNetworking不注意的话
date: 2017-07-16 18:54:10
categories: iOS
tags: 
- objective-c
- iOS
---

## Memory leak 如果你使用AFNetworking不注意的话

之前在开发的过程中，发现app有内存泄露，最终定位到AFNetworking上。但是说什么也不能相信AFNetworking会泄露啊，那么团队在用的一个如此知名的库，怎么会内存泄露呢？所以研究一下AFNetworking的源代码再说。调查后的真相是，这个还真不是AFNetworking的错。因为这个是Foundation库**NSURLSession**的机制导致的。

请先看**NSURLSession**初始化的代码:

```objc
/*
 * Customization of NSURLSession occurs during creation of a new session.
 * If you only need to use the convenience routines with custom
 * configuration options it is not necessary to specify a delegate.
 * If you do specify a delegate, the delegate will be retained until after
 * the delegate has been sent the URLSession:didBecomeInvalidWithError: message.
 */
+ (NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration delegate:(nullable id <NSURLSessionDelegate>)delegate delegateQueue:(nullable NSOperationQueue *)queue;
```
重点是**NSURLSessionDelegate**，通过头文件中方法的注释: 

> if you do specify a delegate, the delegate will be retained until after the delegate has been sent the URLSession:didBecomeInvalidWithError: message.

我们了解到，这个delegate是会被**NSURLSession** retain，只有在delegate的**URLSession:didBecomeInvalidWithError:message** 之后才会被释放。什么时候delegate会被调用这个方法呢？ 答案是**NSURLSession**的:

> invalidateAndCancel和finishTasksAndInvalidate 方法

苹果的官方文档：
> After invalidating the session, when all outstanding tasks have been canceled or have finished, the session calls the delegate's URLSession:didBecomeInvalidWithError: method. When that delegate method returns, the session disposes of its strong reference to the delegate.

再看**AFURLSessionManager**的初始化代码：

```objc
- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration {
    self = [super init];
    if (!self) {
        return nil;
    }

    if (!configuration) {
        configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    }

    self.sessionConfiguration = configuration;

    self.operationQueue = [[NSOperationQueue alloc] init];
    self.operationQueue.maxConcurrentOperationCount = 1;

    self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
...以下省略
```

**AFURLSessionManager**直接使用self初始化**NSURLSession**的，所以如果你不调用上述两个方法之一，必然会泄露。

好在AF提供了相应的方法：

```objc
/**
 Invalidates the managed session, optionally canceling pending tasks.

 @param cancelPendingTasks Whether or not to cancel pending tasks.
 */
- (void)invalidateSessionCancelingTasks:(BOOL)cancelPendingTasks;
```

所以，在使用完**AFURLSessionManager**或者**AFHTTPSessionManager**，请一定记得调用这个方法，不然就会产生内存泄露! 

众所周知，调查内存泄漏的问题不是那么容易，尤其是app非常复杂的时候。希望能帮到看到这篇文章的读者，避免走一些弯路。
