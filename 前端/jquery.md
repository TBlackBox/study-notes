# 一些常用的函数

## siblings()
返回带有类名 "start" 的每个 <li> 元素的所有同级元素：
```
$(document).ready(function(){
$("li.start").siblings().css({"color":"red","border":"2px solid red"});
});
```

结果
```
ul (parent)
    li (sibling)
    li (sibling)
    li (sibling with class name "start")
    li (sibling)
    li (sibling)
```


## 监听滚动条的位置
```
 $(document).scroll(function() {
        var scroH = $(document).scrollTop();  //滚动高度
        var viewH = $(window).height();  //可见高度 
        var contentH = $(document).height();  //内容高度
 
        if(scroH >100){  //距离顶部大于100px时
 
        }
        if (contentH - (scroH + viewH) <= 100){  //距离底部高度小于100px
             
        }  
        if (contentH = (scroH + viewH)){  //滚动条滑到底部啦
             
        }  
 
    });
```