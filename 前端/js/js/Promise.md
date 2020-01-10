# 简介
在JavaScript的世界中，所有代码都是单线程执行的。由于这个原因，导致javaScript的所有网络操作，浏览器事件，都必须通过异步的方式执行，异步可以通过回调函数实现。像ajax 样，异步的，成功或失败都要函数来实现，如果函数里面又是异步，那样就会很长一串，就呕心了赛。如果可以`ajax.ifSuccess(成功的处理).ifFail(失败的处理);`这样，ajax只请求，结果在后面来判断就很不错哦，在es6规范中`Promise` 就可以来做这个事情。

1. Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

2. Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。

# 使用方式
1. 可以使用这个方式看浏览器支持`Promise`不
```
'use strict';

new Promise(function () {});
```
2. 例子 
```
const p = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

p.then(function(value) {
  // success
}, function(error) {
  // failure
});

//另外一个写法，一般省略第二个参数，放到catch处理
p.then(function(value) {
  // success
}).catch(err){
  //错误
};

```
可以结合下面的图进行理解
![](img/promise.png)

说明：Promise参数里面传的是一个异步函数，异步函数有两个参数`resolve`表示成功的返回，`reject`表示异常或者失败的，这两个都是函数，由 JavaScript 引擎提供，不用自己部署。在构造这个`promise`对象的时候，将会执行里面的异步函数。执行完就会返回两种结果，要么成功，要么失败，后面就可以通过`.then(回调函数1，回调函数2)`来获取成功的结果，`回调函数1`对应resolve返回，`回调函数2`对应reject返回，`catch(回调函数)`获取异常或者失败的结果。

3. promise更多的功能

+ 执行链路任务，先做任务1，在做任务2，在做任务三，中途一个失败，就到catch里面去
```
job1.then(job2).then(job3).catch(handleError);
```
job1,job2,job3都是promise对象。


+ promise 并行执行异步任务
```
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 500, 'P1');
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 600, 'P2');
});
// 同时执行p1和p2，并在它们都完成后执行then:
Promise.all([p1, p2]).then(function (results) {
    console.log(results); // 获得一个Array: ['P1', 'P2']
});
```
`Promise.all()`可以用来并行执行两个异步任务，

+ 容错执行，只要一个任务返回，就继续向下执行
```
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 500, 'P1');
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 600, 'P2');
});
Promise.race([p1, p2]).then(function (result) {
    console.log(result); // 'P1'
});
```

由于p1执行较快，Promise的then()将获得结果'P1'。p2仍在继续执行，但执行结果将被丢弃。

如果我们组合使用Promise，就可以把很多异步任务以并行和串行的方式组合起来执行。

# 总结
多动脑来理解这个`promise`对象，其实用途很多，这个只是为了学习的笔记，方便自己忘记了的时候理解，这里只是冰山一角，更多看
[阮一峰的讲解](https://es6.ruanyifeng.com/#docs/promise)
[廖雪峰的讲解](https://www.liaoxuefeng.com/wiki/1022910821149312/1023024413276544)。

下面是一些比较牛逼得用法，需细细研究

```
Promise.prototype.then()
Promise.prototype.catch()
Promise.prototype.finally()
Promise.all()
Promise.race()
Promise.allSettled()
Promise.any()
Promise.resolve()
Promise.reject()
Promise.try()
```