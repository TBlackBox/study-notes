相关资料

[FFmpeg+Nginx+RTMP/HLS 快速搭建直播网站 - Crazymagic - 博客园 (cnblogs.com)](https://www.cnblogs.com/crazymagic/articles/14101332.html)

[ffmpeg+nginx+rtmp+web实现视频直播网站_yuyuefan的博客-CSDN博客](https://blog.csdn.net/sha1996118/article/details/79717471)

[winshining/nginx-http-flv-module: Media streaming server based on nginx-rtmp-module. In addtion to the features nginx-rtmp-module provides, HTTP-FLV, GOP cache and VHOST (one IP for multi domain names) are supported now. (github.com)](https://github.com/winshining/nginx-http-flv-module)

# 简介

由于需要，搭建了一个基于nginx+rtmp的直播网站，通过ffmpeg进行推流，可以通过vlc进行拉流和网页拉流两种方式。



需要的环境：

1. centos 
2. ffmpeg(用于推流)和ffplay(用于测试拉流)
3. vlc(用于拉流测试)

# 搭建nginx直播服务器

## 安装nginx 及nginx-http-flv-module模块

1. 由于rtmp 为附加模块，所以安装nginx这里使用源码安装，可以按照自定义模块和最小版本。

2. 按照nginx 依赖

   ```shell
   yum install -y gcc gcc-c++ autoconf pcre pcre-devel make automake
   yum install -y wget httpd-tools vim
   ```

3. 安装 `nginx-http-flv-module`模块

   这里安装nginx-http-flv-module而不是nginx-rtmp-module，主要原因是nginx-http-flv-module是国内开发的，有中文文档，还有就是nginx-rtmp-module的功能都有。

   ```shell
   cd /opt
   wget https://github.com/winshining/nginx-http-flv-module/archive/master.zip
   unzip master.zip
   ```

   [nginx-http-flv-module官方地址](https://github.com/winshining/nginx-http-flv-module)

4. 下载nginx 并解压

   ```shell
   wget http://nginx.org/download/nginx-1.17.6.tar.gz
   tar -zxvf nginx-1.17.6.tar.gz
   ```

5. 编译安装

   进入解压后的nginx根目录

   ```shell
   cd nginx-1.17.6 && ls
   ```

   通过./configure 配置编译参数，并把nginx-http-flv-module模块加入编译参数里

   ```shell
   ./configure --add-module=../nginx-http-flv-module-master
   ```

6. 修改nginx配置文件，防止编译的一些警告中断编译（nginx编译非常严格，遇到意外都会中断）

   ```shell
   vim objs/Makefile
   ```

   打开这个文件，将CFLAGS 后面有一个 `-Werror` 删除即可

7. nginx编译安装

   ```shell
   make && make install
   ```



## 配置nginx

1. 编辑配置文件`vim /usr/local/nginx/conf/nginx.conf`

将下面模块添加到http模块上面

```conf

rtmp {
     server {
          listen 1935;
          chunk_size 4096;
          application live{  #直播1
              live on;
          }
          application hls{  # 直播2
              live on;
              hls on;
              hls_path /usr/local/nginx/html/hls;  #hls存放的路径
          }
          application vod {
              play /opt/video/vod;
          }
      }
 }
```

2. 修改http模块里面的内容

```conf
#配置m3u8播放
location /hls {
    root html;
    index index.html;
    types{
        application/vnd.apple.mpegurl m3u8;
        video/mp2t ts;
    }
}

location / {
    root   html;
    index  index.html index.htm;
}

expires -1;
#添加跨域控制
add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Headers X-Requested-With;
add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
```

3. 创建必要的目录

   ```shell
   mkdir -p /usr/local/nginx/html/hls
   mkdir -p /opt/video/vod
   ```

   如果创建不成功的话，建议把sudo 加到前面测试一下。



4. 测试配置文件是否正常

   ```shell
   /usr/local/nginx/sbin/nginx -t
   ```

5. 启动

   ```shell
   /usr/local/nginx/sbin/nginx
   ```

6. 重启

   ```shell
   /usr/local/nginx/sbin/nginx -s reload
   ```

   



# 推流

ffmpeg下载地址
```
http://www.ffmpeg.org/download.html
```


1. 通过ffmpeg本地文件流

```
ffmpeg -re -i test.mp4 -vcodec libx264 -acodec aac -f flv rtmp://ip/hls1/test
```
-re 已帧率方式推

-i 指定文件

-vcodec 视频编码

-acodec 音屏编码

-f 指定输出格式

2. 推摄像头的流
```
ffmpeg -f dshow -i video="Integrated Camera" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://ip/hls/test
```

# 拉流
1. 通过ffplay拉流
```
ffplay.exe  rtmp://ip/hls/test
```
2. 通过vlc拉流播放
媒体---->打开网路串流---->网路   将下面地址输入即可
```
rtmp://ip/hls/test
```

3. 通过h5播放

测试html,注意，这点的地址也可以是m3u8文件，`http://ip/hls/test.m3u8`前面配置的http服务也就是为了播放m3u8配置的
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>demo</title>
<link href="https://vjs.zencdn.net/7.0.3/video-js.css" rel="stylesheet">
</head>
<body>
<video id="myVideo" class="video-js vjs-default-skin vjs-big-play-centered" controls preload="auto" width="720" height="540" data-setup='{}'>
<source id="source" type="application/x-mpegURL"
src="rtmp://ip/hls/test" >
</video>
</body>
<script src="https://vjs.zencdn.net/7.0.3/video.js"></script>
</html>

```

**注意：**还有一个简单的播放器可以使用

```
https://www.liveqing.com/docs/manuals/LivePlayer.html#%E5%B1%9E%E6%80%A7-property
```



# 总结

这只是一个简单的搭建过程，如果搭建复杂的直播网站，还需要不断的优化和参数配置，健全等等,更多的配置详见`nginx-http-flv-module`官方文档。

