# 简介

http://c.biancheng.net/view/1852.html

QPlainTextEdit 是一个多行文本编辑器，用于显示和编辑多行简单文本。另外，还有一个 QTextEdit 组件，是一个所见即所得的可以编辑带格式文本的组件，以 HTML 格式标记符定义文本格式。

从《[QComboBox](http://c.biancheng.net/view/1849.html)》一节中的代码实现己经看出，使用 QPlainTextEdit::appendPlainText(const QString 函数就可以向 PlainTextEdit 组件添加一行字符串。

QPlainTextEdit 提供 cut()、copy()、paste()、undo()、redo()、clear()、selectAll() 等标准编辑功能的槽函数，QPlainTextEdit 还提供一个标准的右键快捷菜单。