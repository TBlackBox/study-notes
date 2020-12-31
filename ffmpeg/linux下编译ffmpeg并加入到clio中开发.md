# 1.FFmpeg编译

## 1.1.安装yasm

这里我是直接通过ubuntu包安装的，当然也可以通过编译源码来安装。

```html
sudo apt-get install yasm
```

## 1.2.下载FFmpeg

```html
git clone https://git.ffmpeg.org/ffmpeg.git
```

## 1.3.配置、编译FFMPEG

(1) apt-get install yasm   //这儿需要先安装yasm,否则configure会报错

(2) mkdir ffmpeg_install         //创建一个Install目录，存放编译好之后的东东

(3) ./configure --prefix=./ffmpeg_install     //安装到install目录

```html
./configure --prefix=./ffmpeg_install --enable-shared --disable-static --disable-doc  
```

关于FFMPEG的配置参数，我们可以通过下面命令来查看

```html
./configure --help
```

然后执行

```html
make -j16
make install
```

就可以在host目录下找到我们需要的动态库和头文件了

```html
.
├── bin
│   ├── ffmpeg
│   ├── ffprobe
│   └── ffserver
├── include
│   ├── libavcodec
│   ├── libavdevice
│   ├── libavfilter
│   ├── libavformat
│   ├── libavutil
│   ├── libswresample
│   └── libswscale
├── lib
│   ├── libavcodec.so -> libavcodec.so.57.64.101
│   ├── libavcodec.so.57 -> libavcodec.so.57.64.101
│   ├── libavcodec.so.57.64.101
│   ├── libavdevice.so -> libavdevice.so.57.1.100
│   ├── libavdevice.so.57 -> libavdevice.so.57.1.100
│   ├── libavdevice.so.57.1.100
│   ├── libavfilter.so -> libavfilter.so.6.65.100
│   ├── libavfilter.so.6 -> libavfilter.so.6.65.100
│   ├── libavfilter.so.6.65.100
│   ├── libavformat.so -> libavformat.so.57.56.101
│   ├── libavformat.so.57 -> libavformat.so.57.56.101
│   ├── libavformat.so.57.56.101
│   ├── libavutil.so -> libavutil.so.55.34.101
│   ├── libavutil.so.55 -> libavutil.so.55.34.101
│   ├── libavutil.so.55.34.101
│   ├── libswresample.so -> libswresample.so.2.3.100
│   ├── libswresample.so.2 -> libswresample.so.2.3.100
│   ├── libswresample.so.2.3.100
│   ├── libswscale.so -> libswscale.so.4.2.100
│   ├── libswscale.so.4 -> libswscale.so.4.2.100
│   ├── libswscale.so.4.2.100
│   └── pkgconfig
└── share
    └── ffmpeg
```

# 2.使用FFMPEG

上面我们编译完了FFMPEG之后可以去运行以下bin目录下生成的可执行文件

```html
~/tmp/ffmpeg/ffmpeg/host/bin$ ./ffmpeg 
./ffmpeg: error while loading shared libraries: libavdevice.so.57: cannot open shared object file: No such file or directory
```

发现系统提示找不到动态库，可以用

```html
ldd ffmpeg
```

这样编译出来的bin文件里面没有ffplay如果要生成ffplay需要下面两个步骤

1.编译SDL2

安装 libasound2-dev

```html
sudo apt-get install libasound2-dev
```

否则可能会报下面的错误，不能播放声音

```html
SDL_OpenAudio (2 channels, 32000 Hz): No such audio device

SDL_OpenAudio (1 channels, 32000 Hz): No such audio device

No more combinations to try, audio open failed
```

下载SDL2

http://www.libsdl.org/release/SDL2-2.0.5.zip

编译SDL2

```html
unzip SDL2-2.0.5.zip
cd SDL2-2.0.5/
./configure --prefix=/usr/local/
make
sudo make install
```

2.重新配置编译FFMPEG

在执行./configure是添加 --enable-ffplay

```html
./configure --prefix=host --enable-shared --disable-static --disable-doc --enable-ffplay
make
make install
```

这样就会在host/bin目录下生成ffplay了



# 将编译后的文件加入工程代码

1. 将上面生成的文件ffmpeg_install添加到工程更目录下

2. 修改CMakeLists.txt文件 ，加入下面的  需要修改稿studycpp 名字问project 里面的名字

```c
#设置include 和 库的路径
include_directories(ffmpeg_install/include/)
link_directories(ffmpeg_install/lib/)

#这个要放到中间
add_executable(studycpp main.cpp)

target_link_libraries(
        studycpp
        avcodec
        avdevice
        avfilter
        avformat
        avutil
        swresample
        swscale
)

```

在CMakeLists.txt中引入include和lib

其中要使用到CMake的几个方法：

include_directories 用于引入.h文件
link_directories 用于告诉工程lib的位置
target_link_libraries 将lib link 到工程中
其中target_link_libraries的位置要放在add_executable之后。

另外请注意

target_link_libraries的第一个参数是工程name，也就是CMakeLists.txt中的project(FfmpegDemo)。
target_link_libraries的lib名是类似于libavcodec.a中lib*.a中*的位置，也即是说libavcodec.a的lib名是avcodec