# 简介

##### 提供了三种窗口类型，`QMainWindow`，`QWidget`和`QDialog`，三个类都是用来创建窗口的，可以直接使用，也可以继承后再使用



- `QMainWindow`：包含菜单栏，工具栏，状态栏和标题栏。是最常见的窗口形式，通常被用作为主窗口(不能设置布局，使用setLayout()方法，因为它有自己的布局)
- `QDialog`：是对话框窗口的基类。对话框主要用来执行短期任务，或者与用户进行互动，有模态与非模态。这个窗口没有菜单栏，工具栏等
- `QWidget`不清楚窗口用途时，使用QWidget类



QMainWindow继承自QWidget类，拥有它的派生方法和属性

|方法	|描述|
|-----|---|
|addToolBar()	|添加工具栏|
|centralWidget()	|返回窗口中心的一个控件，未设置时返回NULL|
|menuBar()	|返回主窗口的菜单栏|
|setCentralWidget|	设置窗口中心的控件|
|seStatusBar|	设置状态栏|
|statusBar	|获得状态栏对象后，调用状态栏对象的showMessage(message, int timeout = 0)方法，显示状态栏信息，其中第一个参数是要显示的状态栏信息，第二参数是信息停留时间，单位毫秒，默认为0，表示一直显示，一般在主窗口的底部。|
主窗口(QMainWindow)中会有一个控件(QWidget)占位符来占着中心窗口，可以使用setCentralWidget()来设置中心窗口,这就可以解决著主窗口变换的问题。

