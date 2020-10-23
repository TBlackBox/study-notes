## 1.什么是ffmpeg

引用[wiki百科](https://link.jianshu.com?t=http://zh.wikipedia.org/wiki/FFmpeg)的解析.

> FFmpeg是一个自由软件，可以运行音频和视频多种格式的录影、转换、流功能[1](https://link.jianshu.com?t=http://ffmpeg.org/ffmpeg.html)，包含了libavcodec ─这是一个用于多个项目中音频和视频的解码器库，以及libavformat——一个音频与视频格式转换库。

- `ffmpeg`的官网地址是:[https://www.ffmpeg.org/](https://link.jianshu.com?t=https://www.ffmpeg.org/)
- `ffmpeg`的Github项目地址是:[https://github.com/FFmpeg/FFmpeg](https://link.jianshu.com?t=https://github.com/FFmpeg/FFmpeg)

### 1.1 组件

`FFmpeg`项目由以下几部分组成：

- `FFmpeg`视频文件转换命令行工具,也支持经过实时电视卡抓取和编码成视频文件；
- `ffserver`基于`HTTP`、`RTSP`用于实时广播的多媒体服务器.也支持时间平移；
- `ffplay`用 `SDL`和`FFmpeg`库开发的一个简单的媒体播放器；
- `libavcodec`一个包含了所有`FFmpeg`音视频编解码器的库。为了保证最优性能和高可复用性，大多数编解码器从头开发的；
- `libavformat`一个包含了所有的普通音视格式的解析器和产生器的库。

### 1.2 谁在使用`ffmpeg`

- 使用FFMPEG作为内核视频播放器：`Mplayer`，`ffplay`，`射手播放器`，`暴风影音`，`KMPlayer`，`QQ影音`...
- 使用FFMPEG作为内核的Directshow Filter：`ffdshow`，`lav filters`...
- 使用FFMPEG作为内核的转码工具：`ffmpeg`，`格式工厂`...

## 2.如何安装

`FFmpeg`可以在Windows、Linux还有Mac OS等多种操作系统中进行安装和使用。

这篇文章主要介绍其在Windows下面的安装：

- 下载编译好的Windows版本：

  http://ffmpeg.zeranoe.com/builds/

  （与官网同步）

  ![img](http://printf.qiniudn.com/20140730113852.png)

- FFmpeg分为3个版本：`Static`、  `Shared`、 `Dev`

- 前两个版本可以直接在命令行中使用。包含了三个`exe`:`ffmpeg.exe`，`ffplay.exe`，`ffprobe.exe`

- `Static`版本中的`exe`体积较大,那是因为相关的`Dll`都已经编译进`exe`里面去了。

- `Shared`版本中`exe`的体积相对小很多,是因为它们运行的时候还需要到相关的dll中调用相应的功能

- `Dev`版本用于开发,里面包含了库文件`xxx.lib`以及头文件`xxx.h`

## 3.怎么使用

### 3.1 命令行工具的使用

#### 3.11 `ffmpeg.exe`

用于转码的应用程序:

> 一个简单的转码命令 将input.avi转码成output.ts，并设置视频的码率为640kbps



```css
ffmpeg -i input.avi -b:v 640k output.ts  
```

具体用法参考:   [ffmpeg参数中文详细解释](https://link.jianshu.com?t=http://blog.csdn.net/leixiaohua1020/article/details/12751349)
 详细的使用说明（英文）：[http://ffmpeg.org/ffmpeg.html](https://link.jianshu.com?t=http://ffmpeg.org/ffmpeg.html)

#### 3.12 `ffplay.exe`

主要用于播放的应用程序

> 播放test.avi



```css
ffplay test.avi  
```

具体的使用方法可以参考：[ffplay的快捷键以及选项](https://link.jianshu.com?t=http://blog.csdn.net/leixiaohua1020/article/details/15186441)
 详细的使用说明（英文）：[http://ffmpeg.org/ffplay.html](https://link.jianshu.com?t=http://ffmpeg.org/ffplay.html)

#### 3.13 `ffprobe.exe`

ffprobe是用于查看文件格式的应用程序。
 详细的使用说明（英文）：[http://ffmpeg.org/ffprobe.html](https://link.jianshu.com?t=http://ffmpeg.org/ffprobe.html)