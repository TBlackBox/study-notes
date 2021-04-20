##### 和字体设置似的，这玩意是设置颜色的

```python
import sys

from PyQt5.QtGui import QPalette
from PyQt5.QtWidgets import *

class QColorDialogDemo(QWidget):
    def __init__(self):
        super(QColorDialogDemo, self).__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle('QColorDialogDemo')
        layout = QVBoxLayout()

        self.colorButton = QPushButton('选择颜色')
        self.colorButton.clicked.connect(self.getColor)

        self.colorButton1 = QPushButton('选择背景颜色')
        self.colorButton1.clicked.connect(self.getBGColor)

        self.colorLabel = QLabel('hello 测试字体颜色例子')

        layout.addWidget(self.colorButton)
        layout.addWidget(self.colorButton1)
        layout.addWidget(self.colorLabel)

        self.setLayout(layout)

    def getColor(self):
        color = QColorDialog.getColor()
        p = QPalette()
        p.setColor(QPalette.WindowText, color)
        self.colorLabel.setPalette(p)

    def getBGColor(self):
        color = QColorDialog.getColor()
        p = QPalette()
        p.setColor(QPalette.Window, color)
        self.colorLabel.setAutoFillBackground(True)
        self.colorLabel.setPalette(p)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    main = QColorDialogDemo()
    main.show()
    sys.exit(app.exec_())
```

