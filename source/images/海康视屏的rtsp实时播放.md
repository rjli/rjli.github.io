# 实时视屏播放的简单实现

> 通过使用nginx搭建一个简单的服务器，使用ffmpeg推送rtmp实时流，利用video标签实现可以在web端和手机端的浏览器中进行直播。


##  Windows下面搭建基于rtmp的服务器

### 硬件环境

操作系统：windows7旗舰版 

处  理  器：Intel(R) Core(TM)i5-5200 CPU @2.20GHz 2.20 GHz

系统内存： 8GB

系统类型：64位操作系统

###   软件环境及配置

1. 下载 nginx 1.7.11.3 Gryphon，然后解压到对应的目录；
   [下载链接-Gryphon.zip](http://nginx-win.ecsds.eu/download/nginx%201.7.11.3%20Gryphon.zip)

   > 将解压后的目录名:nginx 1.7.11.3 Gryphon改成:nginx-1.7.11.3-Gryphon

   ​

2. 下载服务器状态检查程序 stat.xsl

   [下载地址](https://github.com/arut/nginx-rtmp-module/)

3. 将nginx-rtmp-module-master.zip解压后复制到目录:nginx-1.7.11.3-Gryphon中。

   >  保证stat.xls的目录为:nginx-1.7.11.3-Gryphon\nginx-rtmp-module\stat.xsl

4. 修改conf\nginx-win.conf 配置文件。
   - 添加rtmp服务。

   ```
   rtmp {
    server {
        listen 1935;
        chunk_size 4000;
        application hls{
            live on ;                      
            hls on ; 
            hls_path html/hls;
            hls_fragment 5s;
        }
     }
   }
   ```


- 配置http server

    ```
    server {
        listen       80;
        location /hls {  
            types {  
                application/vnd.apple.mpegurl m3u8;  
                video/mp2t ts;  
            }  
            root html;  
            expires -1;  
        }  
    ```
5. 启动服务器

   在nginx.exe所在的文件夹，按住shift+右键，选中在此处打开命令窗口，进入windows的cmd。输入以下命令启动nginx服务：

   ```
   nginx.exe -c conf\nginx-win.conf
   ```
6. 启动结果
   ![avatar](E:\learn\gitbook\rtsp-hls-online-live\cmd-result.png) 	

7. 直接在浏览器里输入127.0.0.1就可以进入浏览器的欢迎界面
  ![avatar](E:\learn\gitbook\rtsp-hls-online-live\result.png)

##  提供rtmp直播源

在搭建好基于rtmp的服务器之后，需要提供rtmp直播源。这里我们使用的海康视屏的rtsp直播源。根据nginx.conf中的hls_path配置，下面这个命令会向本地的html\hls下面写入ts片段和m3u8文件。

```
ffmpeg -f rtsp  -re -analyzeduration 8000 -probesize 200000 -i "rtsp://admin:hbj3022810@171.121.218.169:2554/h264/ch33/main/av_stream" -strict -2  -c copy -flvflags aac_seq_header_detect -f flv rtmp://localhost/hls/mystream
```

> 注意
>
> 1、这里提供rtmp源的机器不一定和nginx在同一台物理主机上，可以是网络上的另一台机器，只要保证它能与nginx所在的主机建立tcp链接即可。（也就是nginx主机需要开启rtmp服务的监听端口，这里是1935，当然你也可以修改为其他的端口。）  

## 在网页中展示视屏

在nginx-1.7.11.3-Gryphon/html目录下面创建一个live.html。

```
<html>
<head>
    <link rel="stylesheet" href="http://vjs.zencdn.net/5.10/video-js.css">
</head>
    <video id=example-video width=960 height=540 class="video-js vjs-default-skin" controls autoplay=true>
        <source src="hls/mystream.m3u8" type="application/x-mpegURL">
    </video>
    <script src="http://vjs.zencdn.net/5.10/video.js"></script>
   	<script src="https://npmcdn.com/videojs-contrib-hls@^3.0.0/dist/videojs-contrib-hls.js"></script>
   </script>
    <script>
        var player = videojs('example-video');
        player.play();
    </script>
</html>

```

> 如果只是用video标签是无法播放.m3u8的视屏文件的，需要引入videojs-contrib-hls.js。videojs-contrib-hls支持一堆HLS功能

- web端运行效果

  ![avatar](E:\learn\gitbook\rtsp-hls-online-live\web.png)

- 手机端运行效果


  手机端如果与web可以在同一个网络环境中，那么输入对应本机ip地址也是可以查看的，并且支持横屏、竖屏的切换。

  ![avatar](E:\learn\gitbook\rtsp-hls-online-live\app1.png)



