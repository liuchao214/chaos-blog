---
title: gulp升级（3->4）小记

date: 2020-05-26 11:27:16

index_img: /images/webpack.jpeg

banner_img: /images/20191231163323.jpg

tags: [webpack, gulp]
categories: [工程化]
---

#### 问题1. 在执行gulp命令时控制台报错：Task function must be specified。

解决方法：

4.0错误的写法：

```js
    gulp.task('default', ['del'], function () {
    // default task code here
});
```

修改为最新的正确写法：

```js
    gulp.task('default', gulp.series('del', function () {
    // default task code here
}));
```

#### 问题2. Gulp error: The following tasks did not complete: Did you forget to signal async completion?

```js
var gulp = require('gulp');
gulp.task('message', function () {
    console.log("HTTP Server Started");
});
```

解决办法：

1.这种操作方式是用来新建task的，和3.x的用法一样。

```js
var print = require('gulp-print');
gulp.task('message', function () {
    return gulp.src('package.json')
        .pipe(print(function () { return 'HTTP Server Started'; }));
});
```

2. 返回一个Promise 在异步请求机制中，是有一个Promise对象的，它包含了请求的过程中所有内容。如下：

```js
gulp.task('message', function () {
    return new Promise(function (resolve, reject) {
        console.log("HTTP Server Started");
        resolve();
    });
});
```

3. 返回一个回调函数

这个是最简单的一种方法，gulp会自动将这个回调函数作为一个参数返回到任务中，在完成的时候一定要调用这个函数。如下：

```js
gulp.task('message', function (done) {

    console.log("HTTP Server Started");

    done();

});
```

4. 返回一个子进程child process

当我们只是执行一段纯js代码，没有用到node相关的方法时用这个方法

```js
var spawn = require('child_process').spawn;

gulp.task('message', function () {

    return spawn('echo', ['HTTP', 'Server', 'Started'], {stdio: 'inherit'});

});
```

5. 返回一个 RxJS Observable.

如果你是用RxJS 的时候，可以用这个方法。

```js

var Observable = require('rx').Observable;

gulp.task('message', function () {

    var o = Observable.just('HTTP Server Started');

    o.subscribe(function (str) {

        console.log(str);

    });

    return o;

});
```

#### 问题3. 在执行gulp命令时控制台报错：Received a non-Vinyl object in `dest()`。

````
gulp虽然是基于node的，但是它并没有直接使用node中fs模块里的文件系统和流，而是包装了一层vinyl。
vinyl是一个用来描述文件的简单的数据格式。所以gulp中的流其实是一种vinyl流，与我们通常所见的流其实不是一种流。
````

解决方法： gulp.dest()
创建一个用于将 Vinyl 对象写入到文件系统的流 所以vinyl-source-stream必须与gulp4版本保持一致（2.x版本）
