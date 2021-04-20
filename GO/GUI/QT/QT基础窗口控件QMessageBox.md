##### `QMessageBox`常用的消息对话框有以下几类

- 关于对话框
- 错误对话框
- 警告对话框
- 询问对话框
- 消息对话框



##### 关于差异

- 显示的对话框图标不同
- 显示的按钮不同



```python
		text = self.sender().text()
        if text == '显示关于对话框':
            QMessageBox.about(self, '关于', '这是一个关于对话框')
        elif text == '显示消息对话框':
            repaly = QMessageBox.information(self, '消息', '这是一个消息对话框', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
            print(repaly == QMessageBox.Yes)
        elif text == '显示警告对话框':
            QMessageBox.warning(self, '警告', '这是一个警告对话框', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
        elif text == '显示错误对话框':
            QMessageBox.critical(self, '警告', '这是一个错误对话框', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
        elif text == '显示询问对话框':
            QMessageBox.question(self, '警告', '这是一个询问对话框', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
```

