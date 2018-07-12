##  海康视屏获取rtsp

1、登录企业视屏管理平台，系统登录界面如下所示：

![avatar](hk2.png)

>企 业  名  称：虹桥药用
>访 问  地  址：http://www.hik-online.com/rchongqiao
>用户名/密码：admin/hbj3022810



2.登录成功之后的在配置中查找都赢的RTSP对应的端口号，此处为外网端口为2554。

![avatart](E:\learn\gitbook\rtsp-hls-online-live\hk4.png)

3.查看视频编码设置,此处的视频采用H.264编码实现。

![avatar](E:\learn\gitbook\rtsp-hls-online-live\hk5.png)

4.登录9800平台，查看对应的摄像头的通道信息（可省略）

![avatar](E:\learn\gitbook\rtsp-hls-online-live\hk6.png)

> 访问地址：10.1.12.3:8090
>
> 用户名/密码： admin /Rchb2016

5.查看设备型号（可省略）

![avatar](hk7.png)

6.查看系统对应的摄像头的通道号：（可省略）

![avatar](E:\learn\gitbook\rtsp-hls-online-live\hk1.png)



7.得到rtsp地址

**URL规定：**

```
rtsp://username:password@<ipaddress>/<videotype>/ch<number>/<streamtype>
```



>  注：
>
>  1.VLC可以支持解析URL里的用户名密码，实际发给设备的RTSP请求不支持带用户名密码。
>
>  2.videotype:视屏编码格式[h6264/mpeg4]
>
>  3.ch<number> 通道号 （ch33:IP通道1，ch34：IP通道2.../   ch1:模拟通道1, ch2:模拟通道2...）
>
>  4.streamtype:码流类型[主码流（main/av_stream） 子码流（sub/av_stream）]

我们的rtsp取流地址为：

```
rtsp://admin:hbj3022810@171.121.218.169:2554/h264/ch33/main/av_stream
```

8、测试

使用vlc播放器在添加网络流地址中输入上述地址即可播放。



[相关文章]

[最新海康摄像机、NVR、流媒体服务器、回放取流RTSP地址规则说明](https://blog.csdn.net/xiejiashu/article/details/71786187)

[海康大华RTSP取流URL格式](https://wiki.zohead.com/%E6%8A%80%E6%9C%AF/%E9%9F%B3%E8%A7%86%E9%A2%91/%E7%9B%91%E6%8E%A7/%E6%B5%B7%E5%BA%B7%E5%A4%A7%E5%8D%8ERTSP%E5%8F%96%E6%B5%81URL%E6%A0%BC%E5%BC%8F.md)

