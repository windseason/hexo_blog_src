---
title: Objective-C Dynamic binding 研究
date: 2015-09-22 23:30
tags: objective-c
---

## Objective-C Dynamic binding 研究
对于习惯编写静态编程语言(C#/java/C++)的同学，初识Objective-C，可能会跟我有一样的迷惑。什么迷惑呀？请看下面这段oc的代码：

```objc
@interface MyObject : NSObject
+ (id)factoryMethod;
-(NSString*)tell;
@end

@implementation MyObject
+ (id)factoryMethodB { return [[[self class] alloc] init]; }
- (NSString*)tell
{
	return @"I'm MyObject";
}
@end

@interface MyOtherObject : NSObject
-(void)say;
-(NSString*)tell;
@end
@implementation MyOtherObject
-(void)say
{
	NSLog(@"Hello world");
}

-(NSString*)tell
{
	return @"I'm MyOtherObject";
}
@end

void DoSometing()
{
	MyOtherObject *obj = [MyObject factoryMethod]; (1)
	NSLog(@"%@",[obj tell]);                       (2)
	[obj say];                                     (3)
}

```
代码顺利通过编译。调用DoSomething，运行到第(3)行时出错。先来看第（1）行，我打赌这行已经让只熟悉静态语言的程序员们不明白了，因为照理来说，MyObject的factoryMethod返回的是自己的实例，要是用别的语言编写，编译器第(1)行就报错了，因为类型不匹配呀。为什么OC的编译器不报错呢？不急，我们先来看看这个返回值**id**的定义：

> Objective-C provides a special type that can hold a pointer to any
> object you can construct with Objective-C—regardless of class. This
> type is called id, and can be used to make your Objective-C
> programming generic. （大致意思就是id是一个与类型无关的指向任意对象的东东。能够让你OC编程泛型化。）

几个意思呢？其实id就是一个指针，但是它又不同于c/c++里面的void*。void*是指向内存中任意块的未知对象或者未知内容的。id很明确，它就是用来指向OC对象的。（关于void*是否能转换为OC对象这里暂时不讨论）。好了，这就解释了第(1)行的可行性。

运行到第二行的时候，NSLog输出:

> I'm MyObject

它居然正确的调用了MyObject的tell方法！这就要Dynamic binding（动态绑定）知识来解释了：

>Dynamic binding means that when we call a certain object's method, and there are several implementations of that method, the right one is figured out at runtime. In Objective-C, this idea is used, but is also extended - you are permitted to send any message to any object. At first glance this may sound rather dangerous, but in fact this allows you a lot of flexibility. （大意是：OC的动态绑定的意思是OC扩展了Dynamic binding，使编程人员能够被允许发送任何消息给任何对象。咋一听这种扩展非常危险，但是实际上它带来了很大的灵活性！）

由于MyOtherObject*仅仅是声明一个指针，被赋予id。所以第(2)行怎么执行是在运行时决定的，所以[obj tell]也就顺理成章能够毫无错误的运行了。

第(3)行运行的时候，报错:

> unrecognized selector sent to instance 0x100400100

结合上面的知识，很容易判断，因为MyOtherObject有say方法，所以编译器通过。但是运行时，其实指针的内容是MyObject，MyObject并没有声明say方法，所以就报错了。

关于Dynamic binding还有个更有趣的例子，请看：

```objc
@interface LinkedListNode : NSObject
@property id data;
@property LinkedListNode * child;
@end
@implementation LinkedListNode
@end

void DoSometing2()
{
    LinkedListNode *p = [[LinkedListNode alloc] init];
    p.data = [MyOtherObject new];
    
    LinkedListNode *child = [LinkedListNode new];
    child.data = [MyObject new];
    p.child = child;
    
    LinkedListNode *node;
    for (node = p; node != nil; node = node.child) {
	    [[node data] say];
    }
}
```

编译，没任何错误。运行，报的是跟上面(3)一样的错误。同样的原因，怎么办呢？你可能会想，太灵活了，太容易出错了。幸好，有解决的办法：

```objc
    LinkedListNode *p = [LinkedListNode new];
    p.data = [MyOtherObject new];
     
     LinkedListNode *child = [LinkedListNode new];
     child.data = [MyObject new];
     p.child = child;
     
     LinkedListNode *node;
     for (node = p; node != nil; node = node.child) {
            if ([[node data] respondsToSelector:@selector(say)]) {
                [[node data] say];
            }
        }
     }
```
另外，还可以判断是否是某一特定类：

```objc
if ([node isKindOfClass:[MyOtherObject class]]) {
	[[node data] say];
 }
```

上述例子的factoryMethod的返回值id，苹果已经不推荐这么用了。苹果推荐返回instancetype。它跟id用法一样，但是主要是用在alloc, init或者类的工厂方法上的。
它的好处是 编译器能够检测被调用的对象上是否能够反应传入的消息，如果不能，则发出警告。