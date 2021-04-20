# 简介

http://c.biancheng.net/view/1849.html

QComboBox 是下拉列表框组件类，它提供一个下拉列表供用户选择，也可以直接当作一个 QLineEdit 用作输入。QComboBox 除了显示可见下拉列表外，每个项（item，或称列表项）还可以关联一个 QVariant 类型的变量，用于存储一些不可见数据。



# 用法

## QComboBox 的用法

#### 设计时属性设置

QComboBox 主要的功能是提供一个下拉列表供选择输入。在界面上放置一个 QComboBox 组件后，双击此组件

可以进行编辑，如添加、删除、上移、下移操作，还可以设置项的图标

代码添加

```c++
void Widget::on_btnIniItems_clicked()
{ //"初始化列表"按键
    QIcon   icon;
    icon.addFile(":/images/icons/aim.ico");
    ui->comboBox->clear(); //清除列表
    for (int i=0;i<20;i++)
        ui->comboBox->addItem(icon,QString::asprintf("Item %d",i)); //带图标
        //ui->comboBox->addItem(QString::asprintf("Item %d",i)); //不带图标
}
```

