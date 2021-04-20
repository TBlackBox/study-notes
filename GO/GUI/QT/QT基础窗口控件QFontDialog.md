##### 这个控件可以让用户选择所显示文本的字号大小、样式和格式

```python
import sys
from PyQt5.QtWidgets import *

class QFontDialogDemo(QWidget):
    def __init__(self):
        super(QFontDialogDemo, self).__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle('QFontDialogDemo')

        layout = QVBoxLayout()

        self.fontButton = QPushButton('选择字体')
        self.fontButton.clicked.connect(self.getFont)

        self.fontLabel = QLabel('hello 测试字体例子')

        layout.addWidget(self.fontButton)
        layout.addWidget(self.fontLabel)

        self.setLayout(layout)

    def getFont(self):
        font, ok = QFontDialog.getFont()
        if ok:
            self.fontLabel.setFont(font)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    main = QFontDialogDemo()
    main.show()
    sys.exit(app.exec_())
```

