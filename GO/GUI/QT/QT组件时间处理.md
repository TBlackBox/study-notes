# 简介

http://c.biancheng.net/view/1847.html

时间日期是经常遇到的数据类型，[Qt](http://c.biancheng.net/qt/) 中时间日期类型的类如下：

1. QTime：时间数据类型，仅表示时间，如15:23:13。
2. QDate：日期数据类型，仅表示日期，如2017-4-5。
3. QDateTime：日期时间数据类型，表示日期和时间，如2017-03-23 08:12:43。


Qt 中有专门用于日期、时间编辑和显示的界面组件，介绍如下：

1. QTimeEdit：编辑和显示时间的组件。
2. QDateEdit：编辑和显示日期的组件。
3. QDateTimeEdit：编辑和显示日期时间的组件。
4. QCalendarWidget： 一个用日历形式选择日期的组件。



## 日期时间数据与字符串之间的转换

#### 时间、日期编辑器属性设置

在图 1 窗体左上方的“日期时间”Groupbox 中，使用 QTimeEdit、QDateEdit、QDateTimeEdit 组件作为时间、日期、日期时间编辑器；在其右侧，各放置一个 QLineEdit 组件用于字符串显示。

QDateEdit 和 QTimeEdit 都从 QDateTimeEdit 继承而来，实现针对日期或时间的特定显示功能。实际上，QDateEdit 和 QTimeEdit 的显示功能都可以通过 QDateTimeEdit 实现，只需设置好属性即可。

QDateTimeEdit 类的主要属性的介绍如下：

- datetime：日期时间。
- date：日期，设置 datetime 时会自动改变 date，同样，设置 date 时，也会自动改变 datetime 里的日期。
- time：时间，设置 datetime 时会自动改变 time，同样，设置 time 时，也会自动改变 datetime 里的时间。
- maximumDateTime、 minimumDateTime：最大、最小日期时间。
- maximumDate、minimumDate：最大、最小日期。
- maximumTime、minimumTime：最大、最小时间。
- currentSection：当前输入光标所在的时间日期数据段，是枚举类型 QDateTimeEdit::Section。QDateTimeEdit 显示日期时间数据时分为多个段，单击编辑框右侧的上下按钮可修改当前段的值。如输入光标在YearSection段，就修改“年”的值。
- currentSectionIndex：用序号表示的输入光标所在的段。
- calendarPopup：是否允许弹出一个日历选择框。当取值为 true 时，右侧的输入按钮变成与 QComboBox 类似的下拉按钮，单击按钮时出现一个日历选择框，用于在日历上选择日期。对于 QTimeEdit，此属性无效。
- displayFormat：显示格式，日期时间数据的显示格式，例如设置为“yyyy-MM-dd HH:mm:ss”，一个日期时间数据就显示为“2016-11-02 08:23:46”。



它将日期时间数据按照 format 指定的格式转换为字符串。format 是一个字符串，包含一些特定的字符，表示日期或时间的各个部分，表 2 是用于日期时间显示的常用格式符。



| 字符  | 意义                                               |
| ----- | -------------------------------------------------- |
| d     | 天，不补零显示，1-31                               |
| dd    | 天，补零显示，01-31                                |
| M     | 月，不补零显示，1-12                               |
| MM    | 月，补零显示，01-12                                |
| yy    | 年，两位显示，00-99                                |
| yyyy  | 年，4位数字显示，如 2016                           |
| h     | 小时，不补零，0-23 或 1-12 (如果显示 AM/PM)        |
| hh    | 小时，补零2位显示，00-23 或 01-12 (如果显示 AM/PM) |
| H     | 小时，不补零，0-23 (即使显示 AM/PM)                |
| HH    | 小时，补零显示，00-23 (即使显示 AM/PM)             |
| m     | 分钟，不补零，0-59                                 |
| mm    | 分钟，补零显示，00-59                              |
| z     | 毫秒，不补零，0-999                                |
| zzz   | 毫秒，补零 3 位显示，000-999                       |
| AP或A | 使用 AM/pm 显示                                    |
| ap或a | 使用 am/pm 显示                                    |