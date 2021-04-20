# 简介

- QSpinBox 用于整数的显示和输入，一般显示十进制数，也可以显示二进制、十六进制的数，而且可以在显示框中增加前缀或后缀。

- QDoubleSpinBox 用于浮点数的显示和输入，可以设置显示小数位数，也可以设置显示的前缀和后缀。

QSpinBox 和 QDoubleSpinBox 都是 QAbstractSpinBox 的子类，具有大多数相同的属性，只是参数类型不同。在 UI 设计器里进行界面设计时，就可以设置这些属性。 QSpinBox 和 QDoubleSpinBox 的主要属性。

QSpinBox 和 QDoubleSpinBox 都是 QAbstractSpinBox 的子类，具有大多数相同的属性，只是参数类型不同。在 UI 设计器里进行界面设计时，就可以设置这些属性。



| 属性名称           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| prefix             | 数字显示的前缀，例如“$”                                      |
| suffix             | 数字显示的后缀，例如“kg”                                     |
| minimum            | 数值范围的最小值，如 0                                       |
| maximum            | 数值范围的最大值，如 255                                     |
| singlestep         | 单击右侧上下调整按钮时的单步改变值，如设置为 1，或 0.1       |
| value              | 当前显示的值                                                 |
| displaylntegerBase | QSpinBox 特有属性，显示整数使用的进制，例如 2 就表示二进制   |
| decimals           | QDoubleSpinBox 特有属性，显示数值的小数位数，例如 2 就显示两位小数 |


提示一个属性在类的接口中一般有一个**读取函数**和**一个设置函数**，如 QDoubleSpinBox 的 decimals 属性，读取属性值的函数为 int decimals()，设置属性值的函数为 void setDecimals(int prec)。



