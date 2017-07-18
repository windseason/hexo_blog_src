---
title: iOS 内购商品类型
date: 2017-07-16 22:49:27
tags: iOS
categories: iOS
---

## iOS 内购商品类型

由于最近的项目有IAP，所以又重温了一遍iOS IAP的知识。在开发的过程中，发现网络上的各种文档都有，当然最详细的还是苹果官方IAP文档，但是苹果对每种类型介绍太官方(官话)，让刚开始看的人不太容易弄懂每种商品的区别。另外，深刻理解每种商品对后面的开发非常有帮助。所以，我觉得有必要说一下。

IAP的类型一共4种，接下来，我将对每一种进行通俗易懂的解释。

### Consumable products (消费类型内购)

苹果官方解释
> Items that get used up over the course of running your app. Examples include minutes for a Voice over IP app and one-time services such as voice transcription.

这种产品类型应该大家都知道，很好懂。但是如果仅根据苹果的两个例子，这两个例子都是跟语音通话相关，一个没使用过任何内购的人还真不好理解。通俗来讲，就好比你去菜市场里边，你花X元买了Y个苹果，然后这个交易就完成了。

该类型的内购的例子很多，比如你玩游戏的时候，花了X元买了N个钻石；花X元兑换了N个虚拟币用于购买app中的某些虚拟服务等。

顾名思义，这种类型的商品的特点就是商品可消耗，可重复购买。每次购买的值一般都会叠加。如果买了后，用户不消耗，则一直存在用户相关的账号中。

### Non-consumable products (非消费类型内购)

该种类型的商品主要用于解锁app上的一些功能，或者游戏的某个关卡，又或者是获得某项主题之类的。总之就是当用户购买后，这个商品就一直生效，不需要重复购买。

### Auto-renewable subscriptions (自动续费类型的内购)

这种类型的商品有点跟Non-consumable商品类似，目的都是用户付钱后提供app中特定的某些功能。最大的不同是，该类商品有是有期限的。比如，某些app的VIP系统，这种系统一般按月计算，需要用户按照价格每月扣费。只有用户续费后，VIP才生效并且VIP相关的特权才能够生效。过期不续费，则VIP失效。

该类商品与Non-consumable products的区别是

- 有过期时间
- 可重复购买(系统自动续订)
- 该类商品的内容一般都会不断的更新

与Consumable products的区别是

- 这种商品买了就等于开始消耗，不用用户主动进行后续操作
- 有过期时间
- 苹果系统自动帮用户续费购买


### Non-renewable subscriptions

这种类型的商品跟Auto-renewable subscriptions的商品非常像，目的也是为了给用户提供持续服务的类型。最大的区别是，与该商品相关的续订的信息都由你的服务器保存和处理。简而言之，就是如果你选了Auto-renewable subscriptions，则苹果帮你搞定一切，包括用户的自动续订。但如果你选了Non-renewable subscriptions做为你提供的商品的类型，则你需要负责续订以及过期时间的管理，以及让用户购买的商品在用户所有的设备上可用（只要用户正确登录你的服务器）。哦，对了，还有恢复购买也需要你负责。

