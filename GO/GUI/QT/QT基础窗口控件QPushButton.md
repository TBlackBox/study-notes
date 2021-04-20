# 简介

按钮的基类是QAbstractButton，该类为抽象类，不能实例化，必须由其他按钮类去继承。
常见的按钮类：

- `QPushButton`:最普通辣个按钮
- `QToolButton`: 工具条按钮
- `QRadioButton`：单选按钮，多选一操作
- `QCheckBox`：复选框，复选框可以显示文本或图标。是一种"多选多"的选择。
  除了常用的选中和未选中的两种状态，QCheckBox还提供了第三种状态(半选中)来表明"没有变化"。