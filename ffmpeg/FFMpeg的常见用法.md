## 常见用法

下面介绍 FFmpeg 几种常见用法。

### 1 查看文件信息

查看视频文件的元信息，比如编码格式和比特率，可以只使用`-i`参数。

> ```bash
> $ ffmpeg -i input.mp4
> ```

上面命令会输出很多冗余信息，加上`-hide_banner`参数，可以只显示元信息。

> ```bash
> $ ffmpeg -i input.mp4 -hide_banner
> ```

### 2 转换编码格式

转换编码格式（transcoding）指的是， 将视频文件从一种编码转成另一种编码。比如转成 H.264 编码，一般使用编码器`libx264`，所以只需指定输出文件的视频编码器即可。

> ```bash
> $ ffmpeg -i [input.file] -c:v libx264 output.mp4
> ```

下面是转成 H.265 编码的写法。

> ```bash
> $ ffmpeg -i [input.file] -c:v libx265 output.mp4
> ```

### 4.3 转换容器格式

转换容器格式（transmuxing）指的是，将视频文件从一种容器转到另一种容器。下面是 mp4 转 webm 的写法。

> ```bash
> $ ffmpeg -i input.mp4 -c copy output.webm
> ```

上面例子中，只是转一下容器，内部的编码格式不变，所以使用`-c copy`指定直接拷贝，不经过转码，这样比较快。

### 4.4 调整码率

调整码率（transrating）指的是，改变编码的比特率，一般用来将视频文件的体积变小。下面的例子指定码率最小为964K，最大为3856K，缓冲区大小为 2000K。

> ```bash
> $ ffmpeg \
> -i input.mp4 \
> -minrate 964K -maxrate 3856K -bufsize 2000K \
> output.mp4
> ```

### 4.5 改变分辨率（transsizing）

下面是改变视频分辨率（transsizing）的例子，从 1080p 转为 480p 。

> ```bash
> $ ffmpeg \
> -i input.mp4 \
> -vf scale=480:-1 \
> output.mp4
> ```

### 4.6 提取音频

有时，需要从视频里面提取音频（demuxing），可以像下面这样写。

> ```bash
> $ ffmpeg \
> -i input.mp4 \
> -vn -c:a copy \
> output.aac
> ```

上面例子中，`-vn`表示去掉视频，`-c:a copy`表示不改变音频编码，直接拷贝。

### 4.7 添加音轨

添加音轨（muxing）指的是，将外部音频加入视频，比如添加背景音乐或旁白。

> ```bash
> $ ffmpeg \
> -i input.aac -i input.mp4 \
> output.mp4
> ```

上面例子中，有音频和视频两个输入文件，FFmpeg 会将它们合成为一个文件。

### 4.8 截图

下面的例子是从指定时间开始，连续对1秒钟的视频进行截图。

> ```bash
> $ ffmpeg \
> -y \
> -i input.mp4 \
> -ss 00:01:24 -t 00:00:01 \
> output_%3d.jpg
> ```

如果只需要截一张图，可以指定只截取一帧。

> ```bash
> $ ffmpeg \
> -ss 01:23:45 \
> -i input \
> -vframes 1 -q:v 2 \
> output.jpg
> ```

上面例子中，`-vframes 1`指定只截取一帧，`-q:v 2`表示输出的图片质量，一般是1到5之间（1 为质量最高）。

### 4.9 裁剪

裁剪（cutting）指的是，截取原始视频里面的一个片段，输出为一个新视频。可以指定开始时间（start）和持续时间（duration），也可以指定结束时间（end）。

> ```bash
> $ ffmpeg -ss [start] -i [input] -t [duration] -c copy [output]
> $ ffmpeg -ss [start] -i [input] -to [end] -c copy [output]
> ```

下面是实际的例子。

> ```bash
> $ ffmpeg -ss 00:01:50 -i [input] -t 10.5 -c copy [output]
> $ ffmpeg -ss 2.5 -i [input] -to 10 -c copy [output]
> ```

上面例子中，`-c copy`表示不改变音频和视频的编码格式，直接拷贝，这样会快很多。

### 4.10 为音频添加封面

有些视频网站只允许上传视频文件。如果要上传音频文件，必须为音频添加封面，将其转为视频，然后上传。

下面命令可以将音频文件，转为带封面的视频文件。

> ```bash
> $ ffmpeg \
> -loop 1 \
> -i cover.jpg -i input.mp3 \
> -c:v libx264 -c:a aac -b:a 192k -shortest \
> output.mp4
> ```

上面命令中，有两个输入文件，一个是封面图片`cover.jpg`，另一个是音频文件`input.mp3`。`-loop 1`参数表示图片无限循环，`-shortest`参数表示音频文件结束，输出视频就结束。