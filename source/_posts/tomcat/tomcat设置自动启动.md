---
title: 'Tomcat配置定时启动'
comments: true
date: 2018-07-09 17:19:36
categories: 后端
tags: tomcat java 
---

# Tomcat配置定时启动

## 一、设置tomcat定时启动 

1、首先将tomcat注册为服务      先打开tomcat的bin目录下service.bat文件，修改下面的值，这是sevvice的注册名称和显示名称，一般可使用默认值。

 ```
 set SERVICE_NAME=Tomcat-DPlatform-Server``set DISPLAYNAME=Apache Tomcat 8.5 %SERVICE_NAME%
 ```

​     然后修改jvm大小， 搜索到--JvmMs 128 --JvmMx 256 进行修改，因为做成服务启动，启动的时候就不会用到 catalina.bat，也就不会读取里面的jvm设置了。】（可选，可以不设置）
     然后运行cmd命令窗口，cd 到tomcat的bin目录下，运行下面的命令 

```
service.bat install 
```

​       运行成功过后，会提示服务已经安装成功。如下图所示：
 ![img](file:///C:/Users/cheryl/Documents/My Knowledge/temp/3876c524-2e1d-4692-b26e-4642dd15485c/128/index_files/92bc70f4-85a4-4bc5-87db-9fb697dc3d20.png)
2、制作重启脚本restart.bat，文件内容如下： 

```
@echo off 
echo ***********************************************
echo start time %DATE% %TIME% 
echo ***********************************************
echo Start closing the service 
net stop Tomcat-DPlatform-Server
echo service has been closed
echo  *** 
echo start clean up catching
rd /q/s "D:\tomcat\ydkq-tomcat-server\work\Catalina"
echo clean up end 
echo start clean up temp 
for /f "delims=" %%a in ('dir /ad/b/s D:\tomcat\ydkq-tomcat-server\temp') do (rd /q /s "%%a")>nul
rem del /q/s "D:\tomcat\ydkq-tomcat-server\temp\*.*"   
echo clean up end 
echo Start the start of the service 
net start Tomcat-DPlatform-Server 
echo service has been started
echo ***********************************************
echo end time %DATE% %TIME% 
echo ***********************************************
```

3、配置定时任务
    （win7）开始-->附件——>系统工具——>任务计划与程序，然后设置对应的脚本运行时间计划   【注意】 使用net stop / net start 命令的时候需要使用管理员权限也就是任务中的最高权限，否则会提示发生系统错误，拒绝访问。

4、删除服务

 如果想要删除服务，也很简单，先把服务停掉，然后在cmd窗口运行下面的命令即可，后面那个Tomcat7是服务名。 sc delete Tomcat7

需要注意的是，需要先把服务停掉，才能一次删除成功，或者删除之后再停止服务，就会发现服务已经删除成功了。

【参考】 <http://blog.csdn.net/lovelong8808/article/details/52052423>

