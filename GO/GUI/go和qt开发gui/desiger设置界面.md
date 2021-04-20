# 简介

可以使用qt 带的desiger 设置界面，therecipe/qt提供了uic工具，可以将desiger设计的界面.ui文件转换为go文件为我们使用



## 使用方式

1. 将go 工程目录下建ui包

2. 将ui文件放到包里面

3. 通过下面命令打包，main.go为主入口。打包完成后，在.ui文件的同目录下就会有对应的go 文件。

   ```shell
   qtdeploy build desktop main.go
   ```

   

