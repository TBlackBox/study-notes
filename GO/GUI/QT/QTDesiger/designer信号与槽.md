# 简介

###### 信号(signal)和槽(slot)是Qt的核心机制。

- 信号：是对象或空间发射出去的消息。(可以理解为事件)
- 槽：接收信号。本质是一个函数或方法。(可以理解为事件函数)
  一个信号可以和多个槽来绑定，一个槽可以接收多个信号。



# 方法

- 拖一个button出来
- 点击Edit，找到Edit Signal/slot，点它
- 鼠标左键按住button，朝着你喜欢的方向拖动(也就是朝你控制的方向拖动)，出现这样的红色箭头就松开
- 松开后点击左侧的clicked，然后点击右侧的close(右侧为空的话将下方的复选框勾上)（是一个信号和槽函数选择的兑换框）
- 点击ok，esc退出，Ctrl+r预览，



也可以通过选择edit signal/slot进行设置。