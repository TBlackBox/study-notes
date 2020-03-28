# 常规写法
```
//函数的写法

function run{

 alert("常规写法") //这里是你函数的内容

}

//调用

run()

 
```

# 匿名函数写法
```

var run = function(){

    alert("这是一种声明函数的写法,左边是一个变量,右边是一个函数的表达式,

　　意思就是把一个匿名函数的表达式赋值给了一个变量myrun,只是声明了一个变量指向了一个函数对象")//这里是你函数的内容

}

run()

 
```
# 将方法作为一个对象

```

//作为对象方法,函数写法,这里创建了两个函数外面用{}包裹起来

var Text = {

    run1 : function(){

        alert("这个必须放在一个对象内部,放在外边会出错")//这里是函数内容

    },

    run2 : function(){

        alert("这个必须放在一个对象内部,放在外边会出错")//这里是函数内容

    }

}


Text.run1()//调用第一个函数

Text.run2()//调用第二个函数

```

 # 构造函数中给对象添加方法

 ```

javascript中的每个对象都有prototype属性，Javascript中对象的prototype属性的解释是：返回对象类型原型的引用。






// 给对象添加方法

var funName = function(){};

  funName.prototype.way = function(){

        alert('这是在funName函数上的原始对象上加了一个way方法，构造函数中用到');

    }

    // 调用

    var funname = new text();// 创建对象

    funname.way();//调用对象属性

 ```
# 自执行函数
js自执行函数查到了几种不同写法，放上来给大家看看
## 最前最后加括号 
```

(

function(){alert(1);}()

); 

/*这是jslint推荐的写法，好处是，能提醒阅读代码的人，这段代码是一个整体。 

例如，在有语法高亮匹配功能的编辑器里，光标在第一个左括号后时，最后一个右括号也会高亮，看代码的人一眼就可以看到这个整体。 */

```
## function外面加括号 

```

(function(){alert(1);})(); 

//这种做法比方法1少了一个代码整体性的好处。

```

## function前面加运算符，常见的是!与void 。
```

!function(){alert(1);}(); 

void function(){alert(2);}();
```

