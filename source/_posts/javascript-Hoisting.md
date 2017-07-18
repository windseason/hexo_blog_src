---
title: 什么是Javascript Hoisting？
date: 2017-07-08 00:06:58
categories: javascript
tags: javascript
---

## 什么是Javascript hoisting?

Javascript hoisting 国内一般翻译为**变量提升**。

Hoisting这个词是用来描述Javascript在执行上下文，尤其是创建和执行变量时时如何工作的。但是，hoisting使得Javascript的执行容易被误解。例如，hoisting通常被认为是把变量或者方法在运行时环境被物理地移动到代码的顶部，但是实际上却不是这样。实际上发生的是，变量或者方法的声明在编译时就被提前放入内存，而代码仍然放到原来的位置上。

接下来，通过几个例子来帮助理解

## 方法提升例子
将方法的声明在任何代码执行之前放入内存的好处是能够在方法声明之前就能够使用它，因为它已经被提前放入内存里了。

```javascript
function catName(name) {
   console.log("My cat's name is " + name);
}

catName("Tigger");

/*
运行结果是: "My cat's name is Tigger"
*/
```

上面的代码是一个非常正常的代码。接下来，我们看一下如果把调用放到申明之前会发生什么。

```javascript
catName("Chloe");

function catName(name) {
  console.log("My cat's name is " + name);
}
/*
运行结果是: "My cat's name is Chloe"
*/
```

结果是，跟我们之前写的代码运行的结果一模一样。这是因为Javascript就是这么运行的。

## 变量提升例子
```javascript
num = 6;
num + 7;
var num; 
/* 只要num声明过，无论它在什么位置上，这个代码都运行正常*/
```

**注意**: Javascript 只提升变量的声明，而不提升变量的定义。如果你在声明和定义之前试图获得变量的值，你会得到undefined。

```javascript
var x = 1; // 初始化 x
console.log(x + " " + y); // '1 undefined'，因为定义不会被提升
var y = 2;


// 下面这段代码的结果也是一样的
var x = 1; // Initialize x
var y; // Declare y
console.log(x + " " + y); // '1 undefined'
y = 2; // Initialize y
```
