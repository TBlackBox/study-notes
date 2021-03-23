# 简介
break默认是结束当前循环，但我们多成循环时，想通过内层循环里的语句直接跳出外层循环，怎么做诶？
java提供了使用break直接跳出外层循环，此时需要在break后通过标签指定外层循环。java中的标签是一个紧跟着英文冒号的标识符，与其他语言不同的是，java中的标签只有放在循环语句之前才有作用。
需要注意的是，break后标签必须是一个有效的标签，即这个标签须在break语句所在循环的外层循环之前定义。

# 案列
```java
outer:for(int i =0; i < 3; i++) {
    for(int j = 0; j < 3; j++) {
        if(i == 1) {
        	break outer;
        }else {
        	System.out.println("i=" + i + ", j=" + j);
        }
    }
}
```
输出：
```java
i=0, j=0
i=0, j=1
i=0, j=2
```
这里的`outer`换成其他的都可以，如果不使用`outer`标签的话，则是下面的效果
```java
for(int i =0; i < 3; i++) {
    for(int j = 0; j < 3; j++) {
        if(i == 1) {
            break;
        }else {
            System.out.println("i=" + i + ", j=" + j);
        }
    }
}
```
```java
i=0, j=0
i=0, j=1
i=0, j=2
i=2, j=0
i=2, j=1
i=2, j=2
```
明显多了i为2的3个。同理 continue是一样的用法
```java
outer : for(int i =0; i < 3; i++) {
			for(int j = 0; j < 3; j++) {
				if(j == 1) {
					continue outer;
				}else {
					System.out.println("i=" + i + ", j=" + j);
				}
			}
}
```
输出
```java
i=0, j=0
i=1, j=0
i=2, j=0
```
不使用标签
```java
for(int i =0; i < 3; i++) {
    for(int j = 0; j < 3; j++) {
        if(j == 1) {
            continue;
        }else {
            System.out.println("i=" + i + ", j=" + j);
        }
    }
}

```
输出：

```java
i=0, j=0
i=0, j=2
i=1, j=0
i=1, j=2
i=2, j=0
i=2, j=2
```

# 总结
对比着看，都能瞬间明白outer标签的作用。