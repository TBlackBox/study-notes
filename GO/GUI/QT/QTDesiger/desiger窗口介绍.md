# 简介

QT Designer是专门用来做界面的，生成.ui文件，python 和go 都能通过这个文件生成对应的代码，用于复杂界面的设计。



# 主要区域

最常用的是Widget(窗口)和Main Window(主窗口)。选择Main Window，创建一个主窗口，主窗口可以添加菜单栏和工具栏。

# **Widget Box（工具箱）**

这个里面提供了很多的控件，可以直接拖放。ctrl+r 就可以预览效果了。



# **Object Inspector 对象查看器**

可以查看各个对象的列表和包含关系



# Property Editor 属性编辑器

其中提供了对窗口、控件、布局的属性编辑功能

objectName，控件对象名称
geometry，相对坐标系
sizePolicy，控件大小策略
minimumSize，最小宽度，高度
maximumSize，最大宽度，高度
font，字体
cursor，光标
windowTitle，窗口标题
windowsIcon/icon，窗口图标/控件图标
iconSize，图标大小
toolTip，提示信息
statusTip，任务栏提示信息
text，控件文本
shortcut，快捷键

# Signal/Slot/Editor 信号/槽编辑器、动作编辑器和资源浏览器

其中在信号/槽编辑器中，可以为控件添加自定义的信号和槽函数，编辑控件的信号和槽函数，在资源浏览器中，可以为控件添加图片，比如Label,Button的背景图片