
# 简介
在es6中  要实现请求的次序进行，可以通过async和await配合使用。

优点
1. async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果
2. async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。


# 使用方法
。首先在 function 前面加 async 用来说明这个函数是一个异步函数，当然，在调用函数前加await即可。

# 例子
```

function 函数1(){
        //执行fetch数据请求
        return "数据"
}

async function 函数2（）{
        let data = await 函数1();
        //接着执行
}

```

这样调用函数2的时候，就可以就会等到函数1执行返回后才会继续执行。需要注意的是，函数2返回的是一个promise对象。

# 注意
1. await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。
```
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

2. await命令只能用在async函数之中，如果用在普通函数，就会报错。