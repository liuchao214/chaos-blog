---
title: 实现Promise、Promise.all
date: 2019-01-16 19:08:24
index_img: /images/code.png
banner_img: /images/code-bg.jpg
tags: [JavaScript]
categories: [手撕代码]
---
### 实现Promise
```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

const NewPromise = function(executor) {
  const _this = this;
  _this.status = PENDING;
  _this.data = undefined;
  _this.onResolvedCallback = [];
  _this.onRejectedCallback = [];

  // 成功
  function resolve(value) {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(function() { // 异步执行所有的回调函数
      if (_this.status === PENDING) {
        _this.status = FULFILLED;
        _this.data = value;
        _this.onResolvedCallback.forEach(callback => callback(value));
      }
    })
  }
  // 失败
  function reject(reason) {
    setTimeout(function() { // 异步执行所有的回调函数
      if (_this.status === PENDING) {
        _this.status = REJECTED;
        _this.data = reason;
        _this.onRejectedCallback.forEach(callback => callback(reason));
      }
    })
  }

  try {
    executor(resolve, reject);
  } catch (error) {
    reject(error);
  }
}  
// then
NewPromise.prototype.then = function(onResolved, onRejected) {
  const _this = this;
  if ( typeof onResolved !== 'function') {
   onResolved = function(value) { return value }
  }
  if ( typeof onRejected !== 'function') {
    onRejected = function(reason) { throw reason }
  }

  // 公共判断
  const common = function (data, resolve, reject) {
    // 考虑到有可能throw，我们将其包在try/catch块里
    try {
      let value = _this.status === FULFILLED
        ? onResolved(data)
        : onRejected(data)
      if( value instanceof Promise) {
        value.then(resolve, reject)
      }
      resolve(value)
    } catch (error) {
      reject(error)
    }
  }
  // 公共判断
  const pendingCommon = function (data, flag, resolve, reject) {
    // 考虑到有可能throw，我们将其包在try/catch块里
    try {
      let value = flag === FULFILLED
        ? onResolved(data)
        : onRejected(data)
      if( value instanceof Promise) {
        value.then(resolve, reject)
      }
      resolve(value)
    } catch (error) {
      reject(error)
    }
  }

  if (_this.status === PENDING) {
    return new NewPromise(function(resolve, reject) {
      _this.onResolvedCallback.push((value) => {
        pendingCommon(value, FULFILLED, resolve, reject);
      })

      _this.onRejectedCallback.push((reason) => {
        pendingCommon(reason, REJECTED, resolve, reject);
      })
    })
  } else { // resolve / reject
    return new NewPromise(function (resolve, reject) {
      setTimeout(function () {
        common(_this.data, resolve, reject)
      })
    })
  }
}
NewPromise.resolve = function(value) {
  if (value instanceof Promise) return value;
    if (value === null) return null;
    // 判断如果是promise
    if (typeof value === 'object' || typeof value === 'function') {
        try {
            // 判断是否有then方法
            let then = value.then;
            if (typeof then === 'function') {
                return new NewPromise(then.call(value)); // 执行value方法
            }
        } catch (e) {
            return new NewPromise( (resolve, reject) =>{
                reject(e);
            });
        }
    }
    return new NewPromise( (resolve, reject) =>{
      resolve(value);
  });
}
NewPromise.reject = function(value) {
  if (value instanceof Promise) return value;
    if (value === null) return null;
    // 判断如果是promise
    if (typeof value === 'object' || typeof value === 'function') {
        try {
            // 判断是否有then方法
            let then = value.then;
            if (typeof then === 'function') {
                return new NewPromise(then.call(value)); // 执行value方法
            }
        } catch (e) {
            return new NewPromise( (resolve, reject) =>{
                reject(e);
            });
        }
    }
    return new NewPromise( (resolve, reject) =>{
      reject(value);
  });
}
// catch方法
NewPromise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}
// finally
NewPromise.prototype.finally = function (fun) {
  return this.then((value) => {
        return NewPromise.resolve(fun()).then(() => value);
    }, (err) => {
      return NewPromise.resolve(fun()).then(() => {
        throw err;
    });
  });
};
// defered
NewPromise.deferred = function() {
  const dfd = {}
  dfd.promise = new NewPromise(function(resolve, reject) {
    dfd.resolve = resolve
    dfd.reject = reject
  })
  return dfd
}
// all
NewPromise.all = function(promises) {
  return new NewPromise((resolve, reject) => {
    const result = []
    promises = Array.isArray(promises) ? promises : []
    const len = promises.length;
    if(len === 0) resolve([]);
    let count = 0
    for (let i = 0; i < len; ++i) {
      if(promises[i] instanceof Promise) {
          promises[i].then(value => {
            count ++
            result[i] = value
            if (count === len) resolve(result)
          }, reject)
      } else {
        result[i] = promises[i]
      }
    }
  }) 
}
// race
NewPromise.race = function (promises) {
  return new NewPromise((resolve, reject) => {
    promises = Array.isArray(promises) ? promises : []
    promises.forEach(promise => {
      promise.then(resolve, reject)
    })
  })
}
// allSettled
NewPromise.allSettled = function (promises) {
  return new NewPromise((resolve, reject) => {
    promises = Array.isArray(promises) ? promises : []
    let len = promises.length;
    const argslen = len;
    if (len === 0) return resolve([]);
    let args = Array.prototype.slice.call(promises);
    function resolvePromise(index, value) {
      if(typeof value === 'object') {
        const then = value.then;
        if(typeof then === 'function'){
          then.call(value, function(val) {
            args[index] = { status: 'fulfilled', value: val};
            if(--len === 0) {
              resolve(args);
            }
          }, function(e) {
            args[index] = { status: 'rejected', reason: e };
            if(--len === 0) {
              reject(args);
            }
          })
        }
      }
    }
 
    for(let i = 0; i < argslen; i++){
      resolvePromise(i, args[i]);
    }
  })
}

// 测试
// const promise = new NewPromise(function(resolve, reject) {
//   console.log('ss', 11)
//   // resolve(123)
//   reject('errr')
//   // throw 'ree'
// })
// promise.catch(val => {
//   console.log('val', val)
// })
// const promise1 = new NewPromise(function(resolve, reject) {
//   console.log('ss', 11)
//   resolve(123)
//   // reject('errr')
// })
// const rejected = NewPromise.reject(-1);
// rejected.catch(val => {
//     console.log('rejected', val)
//   })
// console.log('resolved', resolved)
const resolved = NewPromise.resolve(1);
const rejected = NewPromise.reject(-1);
const resolved1 = NewPromise.resolve(17);

const p = NewPromise.all([resolved, resolved1, rejected]);
p.then((result) => {
  console.log('result', result)
}).catch(err => {
  console.log('err', err)
})
```

### 实现Promise.all
```js
Promise.myall = function (arr) {
    return new Promise((resolve, reject) => {
        if (arr.length === 0) {
            return resolve([])
        } else {
            let res = [],
                count = 0
            for (let i = 0; i < arr.length; i++) {
                // 同时也能处理arr数组中非Promise对象
                if (!(arr[i] instanceof Promise)) {
                    res[i] = arr[i]
                    if (++count === arr.length)
                        resolve(res)
                } else {
                    arr[i].then(data => {
                        res[i] = data
                        if (++count === arr.length)
                            resolve(res)
                    }, err => {
                        reject(err)
                    })
                }

            }
        }
    })
}
```

### 实现Promise.race
```js
Promise.myrace = function (arr) {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < arr.length; i++) {
            // 同时也能处理arr数组中非Promise对象
            if (!(arr[i] instanceof Promise)) {
                Promise.resolve(arr[i]).then(resolve, reject)
            } else {
                arr[i].then(resolve, reject)
            }
        }
    })
}
```
