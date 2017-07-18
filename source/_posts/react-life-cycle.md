---
title: React Component lifecycle
date: 2017-07-18 12:17:55
categories: 
- javascript
- react
tags: 
- react
- javascript
---

## React Component lifecycle

最近刚开始使用react-native做项目，有一些心得，所以在这里发表一下。对没有接触过的技术和开发环境，了解核心component的生命周期是非常重要的。能够保证在写代码的过程中不容易犯错。

### 什么是component？
根据React官方文档，**React.component**是一个抽象基类。能够让使用者将UI拆成各个独立地，可重用的component。常见的使用场景是继承它并提供一个**render()**方法。

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

## React lifecycle methods

![](http://ot51d7lis.bkt.clouddn.com/React%20component%20life%20cycle2.png)

上面这张图展示了一个React component的声明周期，从创建到销毁。接下来将逐步介绍以上各个方法并阐述什么时候使用使用它们。

> Mount单词，字典上的意思是："to get up on something above the level of the ground; especially:to seat oneself (as on a horse) for riding" 后面这段意思很形象，骑上马，可以跑了 :[


### componentWillMount

```javascript
componentWillMount()
```

当这个方法被调用的时候，说明你的component将要出现在屏幕上。渲染方法**render**将要被调用。在这个方法内你能干什么呢？

答案是，你可能不能干太多的事情。因为当**componentWillMount**被调用的时候，还没有任何component能够使用，所以你也无法对DOM做任何事情。

另外，大部分初始化已经被component的**constructor**完成了(构造函数是你初始化默认值的最佳地方)

说到这里，你可能觉得这个方法真没用。但是，这里有一个例外场景，就是当你要在**运行时**为整个app**初始化全局配置**的时候, 并且要保证第一时间初始化方法被调用，这个方法是有特别有用的。这就意味着你的component中几乎99%以上都不会用到这个方法。

你可能会看到有人在这个方法里面启动AJAX calls或者执行一个Promise函数。不要这样做，下面会详细讲述原因。

> *常用场景*：需要在运行时确定的App相关的配置
> *是否能够使用setState*: 否

### componentDidMount

```javascript
componentDidMount()
```

这个方法运行时，你的component已经加载上并已可使用。在这个方法里面你可以做很多事情，例如：

- 在一个已经渲染好的画板上绘制新的元素
- 添加event listeners
- 加载数据。为什么这个方法里面适合加载数据呢？因为你无法保证一个异步的请求在component加载完成之前完成，它是不确定的。在某些情况下，还可能会在component unmount之后请求的response才返回回来，而这个时候如果根据response来设置state的话就会产生异常。使用componentDidMount至少可以保证在结果返回之前是有component可以使用的（但是，如果response返回时不进行处理的话，也有可能会发生错误。本文提供一种ES6 promise的解决方案，请看[make cancelable promise](#cancelablePromise)）

基本上，你可以做任何事情

> *常用场景*：发起Ajax请求来加载数据
> *是否能够使用setState*: 是


### componentWillReceiveProps

```javascript
componentWillReceiveProps(nextProps)
```

这个方法被用来接收新的props。也许一些数据在parent component中加载并被传到当前compoent中。

当component能够对新的props做出任何动作之前，componentWillReceiveProps会被调用。

```javascript
componentWillReceiveProps(nextProps) {
	if(nextProps.text !=== this.props.text) {
		this.setState({text: nextProps.text});
	}
}
```

通过代码可以看到，在该方法中，我们可以同时访问当前props和nextProps。所以我们要做的就是：

- 检查props是否有变化
- 如果有变化，就做出相应的变化

**注意**

- React也许会在props没有变化的时候调用该方法。所以总是判断props是否变化是有必要的。这种情况可能发生在parent component引起当前组件re-render。
- React不会在component初始化，设置初始props的调用该方法
- 调用**this.setState**通常不会触发该方法

> *常用场景*：针对props的变化做出相应的状态改变
> *是否能够使用setState*: 是

### shouldComponentUpdate

```javascript
shouldComponentUpdate(nextProps, nextState)
```

使用这个方法告诉React当前state或者props的变化是否会引起re-render。默认情况下，这个方法总是返回true，即需要re-render。如果你想要避免一些无意义或者纯属浪费的re-render，这个方法是你最好的选择。

P.S 根据react最新官方文档，目前，如果**shouldComponentUpdate**方法返回false,则componentWillUpdate(), render() 和 componentDidUpdate()方法都不会被调用。但是，在未来的react版本中，** shouldComponentUpdate**的返回值只具有参考价值而不是强制性的，所以方法返回false后component也有可能会re-render。

> *常用场景*：控制component是否需要re-render
> *是否能够使用setState*: NO

### componentWillUpdate

```javascript
componentWillUpdate(nextProps, nextState)
```

这个方法在re-render之前会被调用。这个方法跟** componentWillReceiveProps**类似，只是你不能在该方法里调用this.setState。

如果你的component使用了**shouldComponentUpdate**并且当props变化的时候需要做一些事情，那么**componentWillUpdate**是一个不错的选择。

> *常用场景*：在有shouldComponentUpdate方法的component中配合着用
> *是否能够使用setState*: NO

### componentDidUpdate

```javascript
componentDidUpdate(prevProps, prevState)
```

在component 更新之后，使用这个方法来操作DOM。这个方法也是一个发送网络请求的好的选择之一，前提是需要判断当前props和prevProps相比是否产生了变化（如果props没有变化，则不需要发送网络请求）

> *常用场景*：根据props或者state的变化，对DOM进行相应的变化
> *是否能够使用setState*: YES

### componentDidUpdate

```javascript
componentWillUnmount()
```

这个方法会在component从屏幕上移除和销毁之前调用。在这个方法中，你需要进行任何必要的清理工作，比如取消network request，invalidate timers，remove event listeners或者清除任何在**componentDidMount()**中创建的DOM元素。

> *常用场景*：清理一切需要必须清理的资源，事务或者事件
> *是否能够使用setState*: NO

## <a id='cancelablePromise'>make cancelable promise</a>

使用Promise发起异步请求，等到异步请求返回的时候，component有可能已经unmounted了。这个时候在response的callback尝试setState会引发异常。怎么避免这种情况呢？

感谢@istarkov提供了一种可取消的Promise的方案：

```javascript
const makeCancelable = (promise) => {
  let hasCanceled_ = false;

  const wrappedPromise = new Promise((resolve, reject) => {
    promise.then(
      val => hasCanceled_ ? reject({isCanceled: true}) : resolve(val),
      error => hasCanceled_ ? reject({isCanceled: true}) : reject(error)
    );
  });

  return {
    promise: wrappedPromise,
    cancel() {
      hasCanceled_ = true;
    },
  };
};
```

如何使用：

```javascript
const cancelablePromise = makeCancelable(
  new Promise(r => component.setState({...}}))
);

cancelablePromise
  .promise
  .then(() => console.log('resolved'))
  .catch((reason) => console.log('isCanceled', reason.isCanceled));

cancelablePromise.cancel(); // Cancel the promise
```

