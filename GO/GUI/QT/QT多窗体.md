# 简介

http://c.biancheng.net/view/1874.html

常用的窗体基类是 QWidget、QDialog 和 QMainWindow，在创建 GUI 应用程序时选择窗体基类就是从这 3 个类中选择。

QWidget 直接继承于 QObject，是 QDialog 和 QMainWindow 的父类，其他继承于 QWidget 的窗体类还有 QSplashScreen、QMdiSubWindow 和 QDesktopWidget。另外还有一个类 QWindow，它同时从 QObject 和 QSurface 继承。





## 窗体类的主要特点和用途如下：

- QWidget：在没有指定父容器时可作为独立的窗口，指定父容器后可以作为容器的内部组件。
- QDialog：用于设计对话框，以独立窗口显示。
- QMainWindow：用于设计带有菜单栏、工具栏、状态栏的主窗口，一般以独立窗口显示。
- QSplashScreen：用作应用程序启动时的splash窗口，没有边框。
- QMdiSubWindow：用于为 QMdiArea 提供一个子窗体，用于MDI（多文档）应用程序的设计。
- QDesktopWidget：具有多个显卡和多个显示器的系统具有多个桌面，这个类提供用户桌面信息，如屏幕个数、每个屏幕的大小等。
- QWindow：通过底层的窗口系统表示一个窗口的类，一般作为一个父容器的嵌入式窗体，不作为独立窗体。





# 窗体类重要特性的设置

#### setAttribute()函数

setAttribute() 函数用于设置窗体的一些属性，其函数原型为：

```c++
void QWidget::setAttribute(Qt::WidgetAttribute attribute, bool on = true)
```

枚举类型 Qt::WidgetAttribute 定义了窗体的一些属性，可以打开或关闭这些属性。枚举类型 Qt::WidgetAttribute 常用的常量及其意义。

| 常量                     | 意义                              |
| ------------------------ | --------------------------------- |
| Qt:: WA_AcceptDrops      | 允许窗体接收拖放来的组件          |
| Qt::WA_DeleteOnClose     | 窗体关闭时删除自己，释放内存      |
| Qt::WA_Hover             | 鼠标进入或移出窗体时产生paint事件 |
| Qt:: WAAcceptTouchEvents | 窗体是否接受触屏事件              |

#### setWindowFlags()函数

setWindowFlags() 函数用于设置窗体标记，其函数原型是：

```c++
void QWidget::setWindowFlags(Qt::WindowFlags type)
```

参数 type 是枚举类型 Qt::WindowType 的值的组合，用于同时设置多个标记。

另外一个函数 setWindowFlag() 用于一次设置一个标记，其函数原型为：

```c++
void QWidget::setWindowFlag(Qt::WindowType flag, bool on = true)
```

可单独打开或关闭某个属性。枚举类型 Qt::WindowType 常用的常量值见表 。

| 常量                                                         | 意义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 表示窗体类型的常量                                           |                                                              |
| Qt::Widget                                                   | 这是 QWidget 类的缺省类型。这种类型的窗体，如果它有父窗体，就作为父窗 体的子窗体；否则就作为一个独立的窗口 |
| Qt::Window                                                   | 表明这个窗体是一个窗口，通常具有窗口的边框、标题栏，而不管它是否有父窗体 |
| Qt::Dialog                                                   | 表明这个窗体是一个窗口，并且要显示为对话框（例如在标题栏没有最小化、 最大化按钮)。这是 QDialog 类的缺省类型 |
| Qt::Popup                                                    | 表明这个窗体是用作弹出式菜单的窗体                           |
| Qt::Tool                                                     | 表明这个窗体是工具窗体，具有更小的标题栏和关闭按钮，通常作为工具栏的 窗体 |
| Qt::ToolTip                                                  | 表明这是用于 Tooltip 消息提示的窗体                          |
| Qt::SplashScreen                                             | 表明窗体是splash屏幕，是 QSplashScreen 类的缺省类型          |
| Qt::Desktop                                                  | 表明窗体是桌面，这是 QDesktopWidget 类的类型                 |
| Qt::SubWindow                                                | 表明窗体是子窗体，例如 QMdiSubWindow 就是这种类型            |
| 控制窗体显示效果的常量                                       |                                                              |
| Qt::MSWindowsFixedSizeDialogHint                             | 在 Windows 平台上，使窗口具有更窄的边框，用于固定大小的对话框 |
| Qt::FramelessWindowHint                                      | 创建无边框窗口                                               |
| WindowHint要定义窗体外观定制窗体外观的常量，需要先设置 Qt::Customize |                                                              |
| Qt::CustomizeWindowHint                                      | 关闭缺省的窗口标题栏                                         |
| Qt::WindowTitleHint                                          | 窗口有标题栏                                                 |
| Qt::WindowSystemMenuHint                                     | 有窗口系统菜单                                               |
| Qt::WindowMinimizeButtonHint                                 | 有最小化按钮                                                 |
| Qt::WindowMaximizeButtonHint                                 | 有最大化按钮                                                 |
| Qt::WindowMinMaxButtonsHint                                  | 有最小化、最大化按钮                                         |
| Qt::WindowCloseButtonHint                                    | 有关闭按钮                                                   |
| Qt::Windo wContextHelpButtonHint                             | 有上下文帮助按钮                                             |
| Qt::WindowStaysOnTopHint                                     | 窗口总是处于最上层                                           |
| Qt::WindowStaysOnBottomHint                                  | 窗口总是处于最下层                                           |
| Qt::WindowTransparentForlnput                                | 窗口只作为输出，不接受输入                                   |


Qt::Widget、Qt::Window 等表示窗体类型的常量可以使窗体具有缺省的外观设置，如果设置为 Qt::Dialog 类型，则窗体具有对话框的缺省外观，例如标题栏没有最小化、最大化按钮。

控制窗体显示效果和外观的设置项可定制窗体的外观，例如设置一个窗体只有最小化最大化按钮，没有关闭按钮。

#### setWindowState()函数

setWindowState() 函数使窗口处于最小化、最大化等状态，其函数原型是：

```c++
void QWidget::setWindowState(Qt::WindowStates windowstate)
```

枚举类型 Qt::WindowState 表示了窗体的状态，其取值见表 。

| 常量                 | 意义                                 |
| -------------------- | ------------------------------------ |
| Qt: :WindowNoState   | 正常状态                             |
| Qt: :WindowMinimized | 窗口最小化                           |
| Qt:: WindowMaximized | 窗口最大化                           |
| Qt::WindowFullScreen | 窗口填充整个屏幕，而且没有边框       |
| Qt:: Window Active   | 变为活动的窗口，例如可以接收键盘输入 |

#### setWindowModality()函数

setWindowModality() 函数用于设置窗口的模态，只对窗口类型有用。其函数原型为：

```go
void setWindowModality(Qt::WindowModality windowModality)
```

枚举类型 Qt::WindowModality 的取值意义见表 。

| 常量                 | 意义                                           |
| -------------------- | ---------------------------------------------- |
| Qt::NonModal         | 无模态，不会阻止其他窗口的输入                 |
| Qt::WindowModal      | 窗口对于其父窗口、所有的上级父窗口都是模态的   |
| Qt::ApplicationModal | 窗口对整个应用程序是模态的，阻止所有窗口的输入 |

#### setWindowOpacity()函数

setWindowOpacity() 函数用于设置窗口的透明度，其函数原型如下:

```c++
void QWidget::setWindowOpacity(qreal level)
```

参数 level 是 1.0（完全不透明）至 0.0（完全透明）之间的数。窗口透明度缺省值是 1.0，即完全不透明。