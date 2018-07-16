---
title: 获得运行时指定DOM元素的Style
tags:
  - javascript
categories: javascript
date: 2018-07-16 13:08:42
---


## 获得运行时指定DOM元素的Style

在工作中有时候需要获得指定DOM元素是否具有某种style，内联样式和层叠样式都需要检查，怎么做呢？直接上代码：

```javascript
/**
 * 将字符串中的'-'开始的子字符串去掉'-'并替换为大写字母
 * 例如background-color,会被替换为backgroundColor
 */
function camelize(str) {
    return str.replace(/-(\w)/g, function (strMatch, p1){
        return p1.toUpperCase();
    });
}

/**
 * 获得指定元素的指定style
 * @param elem DOM 元素
 * @param property style的名字
 */
function getStyle(elem, property) {
    if (!elem || !property) {
        console.log('elem or property should not be undefined or null.');
        return null;
    }

    var value = null;
    if(elem.style){
        value = elem.style[camelize(property)];//先获取是否有内联样式
    }
    try {
        // 无内联样式，则获取层叠样式表计算后的样式
        if (!value || value === "") {
            if (document.defaultView && document.defaultView.getComputedStyle) {
                var css = document.defaultView.getComputedStyle(elem,null);
                value = css ? css.getPropertyValue(property) : null;
            }
        }
    } catch (error) {
        console.log(error);
    }
    return value;
}
```

`camelize`方法将传入的style名称中的`-`替换掉，因为内联样式的名称中不包括它。如`getStyle`方法所体现的，优先在内联样式中查找，如果找到就返回，否则调用`defaultView`的`getComputedStyle`方法来获得层叠样式表中的style。

### defaultView 

什么是`defaultView`? 根据[https://developer.mozilla.org/zh-CN/docs/Web/API/Document/defaultView](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/defaultView)所说：

> 在浏览器中，该属性返回当前 document 对象所关联的 window 对象，如果没有，会返回 null。

### getComputedStyle

> Window.getComputedStyle()方法返回一个对象，该对象在应用活动样式表并解析这些值可能包含的任何基本计算后报告元素的所有CSS属性的值。 私有的CSS属性值可以通过对象提供的API或通过简单地使用CSS属性名称进行索引来访问。

参考：[https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle)