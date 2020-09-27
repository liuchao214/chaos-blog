---
title: 实现Symbol
date: 2019-07-09 12:10:25
index_img: /images/code.png
banner_img: /images/code-bg.jpg
tags: [JavaScript]
categories: [手撕代码]
---

## ES6 Symbol特点
ES6 引入了一种新的原始数据类型 Symbol，表示独一无二的值。
* Symbol 函数前不能使用 new 命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。
* instanceof 的结果为 false
* Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述
* 如果 Symbol 的参数是一个对象，就会调用该对象的 toString 方法，将其转为字符串，然后才生成一个 Symbol 值。
* Symbol 函数的参数只是表示对当前 Symbol 值的描述，相同参数的 Symbol 函数的返回值是不相等的。
* Symbol 值不能与其他类型的值进行运算，会报错。
* Symbol 值可以显式转为字符串。
* Symbol 值可以作为标识符，用于对象的属性名，可以保证不会出现同名的属性。
* Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify() 返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名。
* 如果我们希望使用同一个 Symbol 值，可以使用 Symbol.for。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。
* Symbol.keyFor 方法返回一个已登记的 Symbol 类型值的 key。

```js
window.mySymbolMap = {};

const generateName = (() => {
    let postfix = 0;
    return (descString) => {
        postfix++;
        return `${descString}_${postfix}`;
    };
})();

function MySymbol(description) {
    if (this instanceof MySymbol) throw new TypeError(
        'Symbol is not a constructor');

    const descString = description === undefined ? undefined : String(
        description);

    const symbol = Object.create({
        toString() {
            return this.__Name__;
        },
        valueOf() {
            return this;
        }
    });

    Object.defineProperties(symbol, {
        '__Description__': {
            value: descString,
            writable: false,
            enumerable: false,
            configurable: false
        },
        '__Name__': {
            value: generateName(descString),
            writable: false,
            enumerable: false,
            configurable: false
        }
    });

    return symbol;
}

Object.defineProperties(MySymbol, {
    'for': {
        value(description) {
            const descString = description === undefined ? undefined : String(
                description);
            return mySymbolMap[descString]
                ? mySymbolMap[descString]
                : mySymbolMap[descString] = MySymbol(descString);
        },
        writable: true,
        enumerable: false,
        configurable: true
    },
    'keyFor': {
        value(symbol) {
            for (let key in mySymbolMap) {
                if (mySymbolMap[key] === symbol) return key;
            }
        },
        writable: true,
        enumerable: false,
        configurable: true
    }
});

const test = MySymbol(1);
const test1 = MySymbol(1);
const a = {
    [test]: 1,
    [test1]: 12
};
a[MySymbol('test')] = 123;
a[MySymbol('test')] = 1234;
const foo = MySymbol('foo');
const s1 = MySymbol.for('foo');
const s2 = MySymbol.for('foo');
console.log(String(test));
console.log(test === test1);
console.log(a);
console.log(s1 === s1);
console.log(MySymbol.keyFor(s1));
```
