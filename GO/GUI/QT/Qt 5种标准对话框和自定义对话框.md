# 简介

http://c.biancheng.net/view/1870.html

[Qt](http://c.biancheng.net/qt/) 为应用程序设计提供了一些常用的标准对话框，如打开文件对话框、选择颜色对话框、信息提示和确认选择对话框、标准输入对话框等，用户无需再自己设计这些常用的对话框，这样可以减少程序设计工作量。

| 对话框                  | 常用静态函数名称                                             | 函数功能                                                     |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| QFileDialog 文件对话框  | QString getOpenFileName() QStringList getOpenFileNames() QString getSaveFileName() QString getExistingDirectory() QUrl getOpenFileUrl() | 选择打开一个文件 选择打开多个文件 选择保存一个文件 选择一个己有的目录 选择打幵一个文件，可选择远程网络文件 |
| QcolorDialog 颜色对话框 | QColor getColor()                                            | 选择颜色                                                     |
| QFontDialog 字体对话框  | QFont getFont()                                              | 选择字体                                                     |
| QinputDialog 输入对话框 | QString getText() int getlnt() double getDouble() QString getltem() QString getMultiLineText() | 输入单行文字 输入整数 输入浮点数 从一个下拉列表框中选择输入 输入多行字符串 |
| QMessageBox 消息框      | StandardButton information() StandardButton question() StandardButton waming() StandardButton critical() void about() void aboutQt() | 信息提示对话框 询问并获取是否确认的对话框 警告信息提示对话框 错误信息提示对话框 设置自定义信息的关于对话框 关于Qt的对话框 |





# 自定义对话框

http://c.biancheng.net/view/1871.html

自定义对话框的设计一般从 QDialog 继承，并且可以采用UI设计器可视化地设计对话框。对话框的调用一般包括创建对话框、传递数据给对话框、显示对话框获取输入、判断对话框单击按钮的返回类型、获取对话框输入数据等过程。