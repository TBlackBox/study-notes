# 简介

http://c.biancheng.net/view/1846.html

[Qt](http://c.biancheng.net/qt/) 界面设计时使用最多的组件恐怕就是 QLabel 和 QLineEdit 了，QLabel 用于显示字符串，QLineEdit 用于显示和输入字符串。这两个类都有如下的两个函数用于读取和设置显示文字。

```c++
QString text() const
void setText(const QString &)
```

这两个函数都涉及到 QString 类。QString 类是 Qt 程序里经常使用的类，用于处理字符串。QString 类可以进行字符串与数字之间的转换，使用 QLineEdit 就可以实现数字量的输入与输出。



# 其他的数字显示和输入

- QSlider：滑动条，通过滑动来设置数值，可用于数值输入。实例中使用 4 个滑动条输入红、绿、蓝三色和 Alpha 值，然后合成颜色，作为一个 QTextEdit 组件的底色。
- QScrollBar：卷滚条，与 QSlider 功能类似，还可以用于卷滚区域。
- QProgressBar：进度条，一般用于显示任务进度，可用于数值的百分比显示。实例程序中滑动一个Slider，获取其值并更新ScrollBar和ProgressBar。
- QDial：表盘式数值输入组件，通过转动表针获得输入值。
- QLCDNumber：模仿 LCD 数字的显示组件，可以显示整数或浮点数，显示整数时可以不同进制显示。实例程序中转动表盘，获得的值显示在 LCD 组件中。单击“LCD显示进制”的 RadioButton 时，设置 LCD 的显示进制。

## 各组件的主要功能和属性

#### QSlider

QSlider、QScrollBar 和 Qdial 3 个组件都从 QAbstractSlider 继承而来，有一些共有的属性。QSlider 是滑动的标尺型组件，滑动标尺上的一个滑块可以改变值。

基类 QAbstractSlider 的主要属性包括以下几种：

1. minimum、maximum：设置输入范围的最小值和最大值，例如，用红、绿、蓝配色时，每种基色的大小范围是 0〜255，所以设置 minimum 为 0，maximum 为 255。

2. singleStep：单步长，拖动标尺上的滑块，或按下左/右光标键时的最小变化数值。

3. pageStep：在 Slider 上输入焦点，按 PgUp 或 PgDn 键时变化的数值。

4. value：组件的当前值，拖动滑块时自动改变此值，并限定在 minimum 和 maximum 定义的范围之内。

5. sliderPosition：滑块的位置，若 tracking 属性设置为 true，sliderPosition 就等于 value。

6. tracking：sliderPosition 是否等同于 value，如果 tracking=tme，改变 value 时也同时改变 sliderPosition。

7. orientation：Slider 的方向，可以设置为水平或垂直。方向参数是

    

   Qt

   的枚举类型 enum Qt::Orientation，取值包括以下两种：

   - Qt::Horizontal 水平方向
   - Qt::Vertical 垂直方向

8. invertedAppearance：显示方式是否反向，invertedAppearance=false 时，水平的 Slider 由左向右数值增大，否则反过来。

9. invertedControls：反向按键控制，若 invertedControls=tme，则按下 PgUp 或 PgDn 按键时调整数值的反向相反。


属于 QSlider 的专有属性有两个，如下：

1. tickPosition：标尺刻度的显示位置，使用枚举类型 QSliderzTickPosition，取值包括以下 6 种：
   - QSlider::NoTicks：不显示刻度
   - QSlider::TicksBothSides：标尺两侧都显示刻度
   - QSlider::TicksAbove：标尺上方显示刻度
   - QSlider::TicksBelow：标尺下方显示刻度
   - QSlider::TicksLeft：标尺左侧显示刻度
   - QSlider::TicksRight：标尺右侧显示刻度
2. tickInterval：标尺刻度的间隔值，若设置为 0，会在 singleStep 和 pageStep 之间自动选择。

#### QScrollBar

QScrollBar 从 QAbstractSlider 继承而来的，具有 QAbstractSlider 的基本属性，没有专有属性。

#### QDial

QDial 是仪表盘式的组件，通过旋转表盘获得输入值。QDial 的特有的属性包括以下两种：

1. notches Visible：表盘的小刻度是否可见。
2. notchTarget：表盘刻度间的间隔像素值。

#### QProgressBar

QProgressBar 的父类是 QWidget，一般用于进度显示，常用属性如下：

1. minimum、maximum：最小值和最大值。
2. value：当前值，可以设定或读取当前值。
3. textVisible：是否显示文字，文字一般是百分比表示的进度。
4. orientation：可以设置为水平或垂直方向。
5. format：显示文字的格式，“％p%”显示百分比，“％v”显示当前值，“％m”显示总步数。缺省为“％p%”。

#### QLCDNumber

QLCDNumber 是模拟 LCD 显示数字的组件，可以显示整数或小数，但就如实际的 LCD 一样，要设定显示数字的个数。显示整数时，还可以选择以不同进制来显示，如十进制、二进制、十六进制。其主要属性如下：

1. digitCount：显示的数的位数，如果是小数，小数点也算一个数位。
2. smallDecimalPoint：是否有小数点，如果有小数点，就可以显示小数。
3. mode：数的显示进制，通过调用函数 setDecMode()、setBinMode()、setOctMode()、setHexMode() 可以设置为常用的十进制、二进制、八进制、十六进制格式。
4. value：返回显示值，浮点数。若设置为显示整数，会自动四舍五入后得到整数，设置为 intValue 的值。如果 smallDecimalPoint=tme，设置 value 时可以显示小数，但是数的位数不能超过 digitCounto
5. intValue：返回显示的整数值。


例如，若 smallDecimalPoint=tme，digitCount=3，设置 value=2.36，则界面上 LCDNumber 组件会显示为 2.4；若设置 value=1456.25，则界面上 LCDNumber 组件只会显示 145。所以，用 QLCDNumber 作为显示组件时，应注意这些属性的配合。