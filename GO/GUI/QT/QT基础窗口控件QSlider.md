##### QSlider控件是一个滑动条，用于控制有界值，允许水平或竖直滑动。



```python
self.setWindowTitle('滑块控件演示')
self.resize(300, 100)

layout = QVBoxLayout()
self.label = QLabel('hello pyqt5')
self.label.setAlignment(Qt.AlignCenter)

# 设置水平滑动
self.slider = QSlider(Qt.Horizontal)
# 设置最小值
self.slider.setMinimum(12)
# 设置最大值
self.slider.setMaximum(48)
# 设置步长
self.slider.setSingleStep(2)
# 设置当前值
self.slider.setValue(18)
# 设置刻度的位置,设置在下方
self.slider.setTickPosition(QSlider.TicksBelow)
# 设置刻度的间隔
self.slider.setTickInterval(6)
self.slider.valueChanged.connect(self.valueChange)

layout.addWidget(self.slider)
layout.addWidget(self.label)

self.setLayout(layout)
```

