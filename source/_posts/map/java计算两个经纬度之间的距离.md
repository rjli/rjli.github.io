---
title: '计算两个经纬度之间的距离'
comments: true
date: 2017-05-10 10:24:00
categories: 后端
tags: [java,经纬度]
---

#  计算两个经纬度之间的距离

本文使用的是距离公式是有google地图提供的。

### 公式说明

 ![关于经纬度求距离 - cza55007 - NO.1.LY](http://img849.ph.126.net/GNSrkddcKlz3MbknMDS1zA==/2693715527121896352.bmp)

对上面的公式解释如下：
1.Lat1 Lng1 表示A点经纬度，Lat2 Lng2 表示B点经纬度；
2.a=Lat1 – Lat2 为两点纬度之差  b=Lng1 -Lng2 为两点经度之差；
3.6378.137为地球半径，单位为千米；
计算出来的结果单位为千米，若将半径改为米为单位则计算的结果单位为米。
计算精度与谷歌地图的距离精度差不多，相差范围在0.2米以下。

### 代码示例

```
private static final double EARTH_RADIUS = 6378.137;// 地球半径,单位千米
//将角度换算成弧度
private static double rad(double d) {
	return d * Math.PI / 180.0;
}
/**
* 用来比较是否在规定考勤范围
* @param lat1第一个纬度
* @param lng1第一个经度
* @param lat2第二个纬度
* @param lng2第二个经度
* @return 两个经纬度的距离（km）
*/
public static double getDistance(double lat1, double lng1, double lat2,double lng2) {
    double radLat1 = rad(lat1);
    double radLat2 = rad(lat2);
    double a = radLat1 - radLat2;
    double b = rad(lng1) - rad(lng2);
    double s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2)
    + Math.cos(radLat1) * Math.cos(radLat2)
    * Math.pow(Math.sin(b / 2), 2)));
    s = s * EARTH_RADIUS;
    //此处加上double类型转换是因为对于在几百的距离差值之前计算为0，无法达到预期效果
    s = (double)Math.round(s * 10000) / 10000;
    s = s * 10000/ 10000;
    return s;
}
```




顺带提一下百度地图提供的计算两地经纬度的方法，很简单的一句话调用，可以自行去看百度地图API试试，计算结果单位：米

```
var map = new BMap.Map("allmap");
var pointA = new BMap.Point(106.486654,29.490295);  // 点坐标A
var pointB = new BMap.Point(106.581515,29.615467);  // 点坐标B	
```

## 参考文章

- [地理空间距离计算及优化](https://blog.csdn.net/u011001084/article/details/52980834)
- [根据两点经纬度计算距离](https://blog.csdn.net/b_h_l/article/details/8657040)