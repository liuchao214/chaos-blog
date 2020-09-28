---
title: Vue的dom更新机制 & Vue的nextTick
date: 2019-01-09 21:48:52
index_img: /images/Vue-cta.jpg
tags: [Vue]
categories: [框架、库学习]
---
### 前言：
> Vue 实现响应式并不是数据发生变化之后 DOM 立即变化，而是按一定的策略进行 DOM 的更新。
  
> 简单来说，Vue 在修改数据后，视图不会立刻更新，而是等同一事件循环中的所有数据变化完成之后，再统一进行视图更新。
  
> 同步里执行的方法，每个方法里做的事情组成一个事件循环；接下来再次调用的是另一个事件循环。

> nextTick：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，会获取更新后的 DOM。

### $nextTick用法：
```js
//改变数据
vm.message = 'changed';

//想要立即使用更新后的DOM。这样不行，因为设置message后DOM还没有更新
console.log(vm.$el.textContent); // 并不会得到'changed'

//这样可以，nextTick里面的代码会在DOM更新后执行
Vue.nextTick(function() {
    console.log(vm.$el.textContent); //可以得到'changed'
});
```

### 图解：
![image](/images/pages/1234412-20191114161514728-1287582393.png)

上图中第一块：

1 首先修改数据，这是同步任务。同一事件循环的所有的同步任务都在主线程上执行，形成一个执行栈，此时还未涉及 DOM 。

2 Vue 开启一个异步队列，并缓冲在此事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会被推入到队列中一次。

上图中第二块：

同步任务执行完毕，开始执行异步 watcher 队列的任务，更新 DOM 。Vue 在内部尝试对异步队列使用原生的 Promise.then 和 MessageChannel 方法，如果执行环境不支持，会采用 setTimeout(fn, 0) 代替。

上图中第三块：

下次 DOM 更新循环结束之后，此时通过 Vue.nextTick 获取到改变后的 DOM 。通过 setTimeout(fn, 0) 也可以同样获取到。

### 应用场景：
> 需要注意的是，在 created 和 mounted 阶段，如果需要操作渲染后的试图，也要使用 nextTick 方法。
> 官方文档说明：mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 vm.$nextTick：

```
mounted() {
    this.$nextTick(function() {
        // Code that will run only after the
        // entire view has been rendered
    });
}
```

```
showsou() {
    // 显示输入框
    this.showInput = true;
    // 此时事件循环还没有结束，dom没有更新，获取不到input，不能使他产生焦点
    document.getElementById('keywords').focus();
}
```

```
showsou() {
    this.showInput = true;
    this.$nextTick(function() {
        // DOM 更新了
        document.getElementById('keywords').focus();
    });
}
```

```html
<div id="app">
    <p ref="myWidth" v-if="showMe">{{ message }}</p>
    <button @click="getMyWidth">获取p元素宽度</button>
</div>

getMyWidth() {
    this.showMe = true;
    // 以下代码报错 TypeError: this.$refs.myWidth is undefined
    // this.message = this.$refs.myWidth.offsetWidth;

    this.$nextTick(()=>{
        // dom元素更新后执行，此时能拿到p元素的属性
        this.message = this.$refs.myWidth.offsetWidth;
    })
}
```
![image](/images/pages/1234412-20200418162600879-352593538.png)
