# 简介

##### 1、控件的最大值与最小值

一个控件拖到主窗口后可以随意放大或缩小，但是也是有限制的，位置在属性编辑器，属性名为`minimumSize`,`maximumSize`。

**2、尺寸策略(sizePolicy)**
了解尺寸策略之前，先了解sizeHint(期望尺寸)和minisizeHInt(最小期望尺寸)

- sizeHint(期望尺寸)
  每个控件的期望尺寸是不同的，在未设置控件最大值最小值之前，控件推荐到某个尺寸，像默认尺寸一样。但对大多数控件来说，期望尺寸是只读的。
  获取期望值代码如下

```c++
self.控件名.sizeHint().width()
self.控件名.sizeHint().height()
```

- minisizeHInt(最小期望尺寸)
  最小期望尺寸获取，代码和期望尺寸差不多

```python
self.控件名.minimumSizeHint().width()
self.控件名.minimumSizeHint().height()
```



# 水平垂直策略的各种英文解释

Fixed：窗口控件具有其sizeHint所提示的尺寸且尺寸不会再改变；

1. Minimum：窗口控件的sizeHint所提示的尺寸就是它的最小尺寸，该窗口控件不能压缩的比这个值小，但可以变得更大
2. Maximum：窗口控件的sizeHint所提示的尺寸就是它的最大尺寸，该窗口控件不能变得比这个值大，但它可以被压缩到minisizeHint给定的尺寸大小；
3. Preferred：窗口控件的sizeHint所提示的尺寸就是它的期望尺寸，该窗口控件可以缩小到minisizeHint所提示的尺寸，也可以变得比sizeHint所提示的尺寸还大；
4. Expanding：窗口控件可以缩小到minisizeHint所提示的尺寸，也可以变得比sizeHint所提示的尺寸大，但它希望能变得更大；
   MinimumExpanding：窗口控件的sizeHint所提示的尺寸就是它的最小尺寸，该窗口控件不能被压缩得比这个值还小，但它希望能够变得更大；
5. Ignored：无视窗口控件的sizeHint和minisizeHint所提示的尺寸，按照默认来设置。
   Minimum指的是该窗口控件的尺寸不能低于sizeHint;
6. Maximum：指的是该窗口控件不能大于sizeHint。
   