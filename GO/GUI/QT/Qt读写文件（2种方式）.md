# 简介

http://c.biancheng.net/view/1877.html

文件的读写是很多应用程序具有的功能，甚至某些应用程序就是围绕着某一种格式文件的处 理而开发的，所以文件读写是应用程序开发的一个基本功能。

文本文件是指以纯文本格式存储的文件，例如用 [Qt](http://c.biancheng.net/qt/) Creator 编写的 [C++](http://c.biancheng.net/cplus/) 程序的头文件（上文件）和源程序文件（.cpp 文件）。HTML 和 XML 文件也是纯文本文件，只是其读取之后需要对内容进行解析之后再显示。

Qt 提供了两种读写纯文本文件的基本方法：

1. 用 QFile 类的 IODevice 读写功能直接进行读写
2. 利用 QFile 和 QTextStream 结合起来，用流（Stream)的方法进行文件读写。

