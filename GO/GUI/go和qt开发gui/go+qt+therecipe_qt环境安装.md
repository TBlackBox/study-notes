# 简介

使用go 开发gui界面，这里介绍软件的安装和自己踩的坑

电脑

win10  64位

安装的软件需要，严格按照版本需要来配置

```
go的版本
1.13.4  amd/64

qt版本
5.13.2

minggw的版本
7.3  64
```

参考文档

```
https://www.e-learn.cn/topic/3842505

https://zhuanlan.zhihu.com/p/191591015

https://github.com/therecipe/qt/wiki/Installation-on-Windows
```





# go 安装

下载地址

```
https://golang.org/dl/
```

1. 在官网下载对应的版本，安装即可

2. 注意配置GOROOT和GOPATH

- GOROOT: 即按照目录
- GOPATH: 即用户依赖的目录，一般按照go 会在用户变量里面自动添加



3. 环境变量配置好就可以了,在系统的path 里面添加环境变量

   ```
   C:\Go\bin
   ```

   

4. `go version `访问到go的版本



# QT安装

QT安装，分为在线安装和离线安装，这点介绍**在线安装**，安装需要梯子或者国内镜像

1. 下载地址（在线安装）

   ```
   https://download.qt.io/official_releases/online_installers/
   ```

2. 下载下来后 安装即可

   这点安装一定注意，安装比较满，建议不要一次性安装完，不然都会导致中途短线后，所有的等待都是突然，先进性安装，安装完后，在通过安装目录下的`MaintenanceTool.exe`安装需要的qt版本，安装版本位5.13.2，建议安装这个版本，

3. 安装后，配置环境变量，将环境变量配置到系统的path里面

   ```
   C:\Qt\5.13.2\mingw73_64\bin
   C:\Qt\Tools\mingw730_64\bin
   C:\Qt\Tools\mingw730_64\opt\bin
   C:\Qt\Tools\QtCreator\bin
   ```

4. 这个时候 可以使用qtcreater这些了



# 安装therecipe/qt

windows 官方安装介绍

```
https://github.com/therecipe/qt/wiki/Installation-on-Windows
```

这个分两种安装方式，一种是gopath的方式  一种是go mod的方式，这点**介绍gopath,**go mod  我目前没有搞成功，一定要区分好go path  和模块的区别，还有介绍gopath 和goroot的区别。

1. 在用户变量里配置一下qt环境，让therecipe/qt 知道用来那些环境

   ```
   变量				变量值
   QT_DIR    		C:\Qt
   QT_VERSION  	5.13.2
   ```

2. 使用步骤和命令

   注意: GOPATH和GOROOT不是同一个目录!

   ```
   
   go get -v github.com/therecipe/qt
   
   go install -v -tags=no_env github.com/therecipe/qt/cmd/...
   
   for /f %v in ('go env GOPATH') do %v\bin\qtsetup
   ```

   这些命令可以在gopath 下执行，**需要多等一会，不要着急，一步一步做好，不要报错**。

   

   3. 安装好后，会出现3给测试案列，即按照完成



# 测试

新建一个go文件

```go
package main

import (
	"github.com/therecipe/qt/core"
	"github.com/therecipe/qt/widgets"
	"os"
)
func main() {
	core.QCoreApplication_SetApplicationName("sample")
	core.QCoreApplication_SetOrganizationName("sample")
	core.QCoreApplication_SetAttribute(core.Qt__AA_EnableHighDpiScaling, true)
	widgets.NewQApplication(len(os.Args), os.Args)
	window := widgets.NewQMainWindow(nil, 0)
	window.SetMinimumSize2(400, 300)
	window.SetWindowTitle("Hello World")
	window.Show()
	widgets.QApplication_Exec()
}
```

通过`qtdeploy build desktop main.go`运行接口打包一个窗口，这个包在通目录下面的windows 下。

