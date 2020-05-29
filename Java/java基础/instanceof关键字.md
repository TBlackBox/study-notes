多态性带来了一个问题，就是如何判断一个变量所实际引用的对象的类型 。 C++使用runtime-type information(RTTI)，Java 使用 instanceof 操作符。

instanceof 运算符用来判断一个变量所引用的对象的实际类型，注意是它引用的对象的类型，不是变量的类型。

```
if(obj instanceof Object){
	System.out.println("我是一个对象");
}
if(obj instanceof People){
	System.out.println("我是人类");
}
```
如果是object或者People的引用类型或者子类，这返回true,否则返回false.