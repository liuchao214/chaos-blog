---
title: 实现new、bind、instanceOf、deepClone
date: 2017-08-21 16:23:53
index_img: /images/code.png
banner_img: /images/code-bg.jpg
tags: [JavaScript]
categories: [手撕代码]
---
### new 操作符
```js
var New = function(Fn) {
  var obj = {}; // 创建空对象
  var arg = Array.prototype.slice.call(arguments, 1);
  obj.__proto__ = Fn.prototype; // 将obj的原型链__proto__指向构造函数的原型prototype
  obj.__proto__.constructor = Fn; // 在原型链 __proto__上设置构造函数的构造器constructor，为了实例化Fn
  Fn.apply(obj, arg); // 执行Fn，并将构造函数Fn执行obj
  return obj; // 返回结果
};
```

### bind
```js
Function.prototype.bind2 = function(context) {
  if (typeof this !== "function") {
    throw new Error("...");
  }
  var that = this;
  var args1 = Array.prototype.slice.call(arguments, 1);
  var bindFn = function() {
    var args2 = Array.prototype.slice.call(arguments);
    var that2 = this instanceof bindFn ? this : context; // 如果当前函数的this指向的是构造函数中的this 则判定为new 操作。如果this是构造函数bindFn new出来的实例，那么此处的this一定是该实例本身。
    return that.apply(that2, args1.concat(args2));
  };
  var Fn = function() {}; // 连接原型链用Fn
  // 原型赋值
  Fn.prototype = this.prototype; // bindFn的prototype指向和this的prototype一样，指向同一个原型对象
  bindFn.prototype = new Fn();
  return bindFn;
};
```


### instanceOf
```js
const instanceOf = (left, right) => {
  let proto = left.__proto__;
  let prototype = right.prototype;
  while (true) {
    if (proto === null) {
      return false;
    } else if (proto === prototype) {
      return true;
    }
    proto = proto.__proto__;
  }
};
```

### 深拷贝
```js
const getType = data => {
  // 获取数据类型
  const baseType = Object.prototype.toString
    .call(data)
    .replace(/^\[object\s(.+)\]$/g, "$1")
    .toLowerCase();
  const type = data instanceof Element ? "element" : baseType;
  return type;
};
const isPrimitive = data => {
  // 判断是否是基本数据类型
  const primitiveType = "undefined,null,boolean,string,symbol,number,bigint,map,set,weakmap,weakset".split(
    ","
  ); // 其实还有很多类型
  return primitiveType.includes(getType(data));
};
const isObject = data => getType(data) === "object";
const isArray = data => getType(data) === "array";
const deepClone = data => {
  let cache = {}; // 缓存值，防止循环引用
  const baseClone = _data => {
    let res;
    if (isPrimitive(_data)) {
      return data;
    } else if (isObject(_data)) {
      res = { ..._data };
    } else if (isArray(_data)) {
      res = [..._data];
    }
    // 判断是否有复杂类型的数据，有就递归
    Reflect.ownKeys(res).forEach(key => {
      if (res[key] && getType(res[key]) === "object") {
        // 用cache来记录已经被复制过的引用地址。用来解决循环引用的问题
        if (cache[res[key]]) {
          res[key] = cache[res[key]];
        } else {
          cache[res[key]] = res[key];
          res[key] = baseClone(res[key]);
        }
      }
    });
    return res;
  };
  return baseClone(data);
};
```
