# 简介

**基础控件QWidget类是所有用户界面对象的基类，所有的窗口和空间都直接或间接的继承自QWidget**



###### 窗口坐标系统

使用统一的坐标系统来定位窗口控件的位置和大小，以屏幕的左上方角原点，向右为x轴正方向，向下为y轴正方向。
窗口内部也有自己的坐标系统，左上方角原点，向右为x轴正方向，向下为y轴正方向。x轴,y轴围成的区域叫做客户区，客户区的周围是标题栏和边框。



QWidget的成员函数可以分为三类

- QWidget直接提供的成员函数：x()、y()获得窗口左上角的坐标，width()，height()获得客户区的宽度和高度；
- QWidget的geometry()提供的成员函数：x()、y()获得客户区左上角的坐标，width()，height()获得客户区的宽度和高度；
- QWidget的frameGeometry()提供的成员函数：x()、y()获得客户区左上角的坐标，width()，height()获得包含客户区、标题栏和边框在内的整个窗口的宽度和高度；
  

## 常用的几何结构
- 不包含外边各种边框的几何结构
- 包含外边各种边框的几何结构

QWidget不包含边框的常用函数
一般情况下，不包含边框的部分叫做客户区，这部分是一个长方形，在Qt中保存这个长方形使用的是QRect类，改变它的大小位置，使用如下几个函数。

- 改变客户区面积

```
QWidget.resize(width, height)
QWidget.resize(QSize)
```

- 获得客户区大小

```
QWidget.size()
```



- 获得客户区的高度和宽度

  ```
  QWidget.width()
  QWidget.height()
  ```

  

- 设置客户区的宽度和高度

```
QWidget.setFixedWidth(int width)  # 高度固定，只可以改变宽度
QWidget.setFixedHeight(int height)  # 宽度固定，只可以改变高度
QWidget.setFixedSize(QSize size)  # 固定大小，不可以通过鼠标改变
QWidget.setFixedSize(int width, int height)  # 固定大小，不可以通过鼠标改变
QWidget.setGeometry(int x, int y, int width, int height)
QWidget.setGeometry(QRect rect)
```

2. QWidget包含边框的常用函数
- 获得窗口的大小和位置

   ```
   QWidget.frameGeometry()
   ```
- 设置窗口的位置
```
QWidget.move(int x, int y)
QWidget.move(QPoint point)
```
- 获得窗口左上角的坐标
```
QWidget.pos()
```