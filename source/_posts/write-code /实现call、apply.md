---
title: 实现call、apply
date: 2018-09-25 11:00:07
index_img: /images/code.png
banner_img: /images/code-bg.jpg
tags: [JavaScript]
categories: [手撕代码]
---
### 实现call
```js
Function.prototype.mycall = function () {
    let [thisArg, ...args] = [...arguments];
    thisArg = Object(thisArg) || window;
    let fn = Symbol();
    thisArg[fn] = this;
    let result = thisArg[fn](...args);
    delete thisArg[fn];
    return result;
};
```

### 实现apply
```js
Function.prototype.myapply = function () {
    let [thisArg, args] = [...arguments];
    thisArg = Object(thisArg);
    let fn = Symbol();
    thisArg[fn] = this;
    let result = thisArg[fn](...args);
    delete thisArg.fn;
    return result;
};
```
