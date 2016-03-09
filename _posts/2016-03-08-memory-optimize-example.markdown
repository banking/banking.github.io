---
layout: post
title:  "Android 内存优化方案和总结"
date:   2016-03-07 19:57:04 +0800
categories: jekyll update
---

整理一下顺风车4.2.4内存优化案例,和方法总结。  

# 1.首页优化运营图背景

第一次通过Mat分析观察到了异常Bitmap对象,猜测和首页运营图相关。
![place holder](http://121.42.160.4:8081/memoryleak/EntranceFragment_image_exception1.png)
![place holder](http://121.42.160.4:8081/memoryleak/EntranceFragment_image_exception2.png)
qi
可以看到已经定位到了业务代码BtsHomeImageView。继续跟进这个控件，可以看到其父类BezelImageView在初始化时创建了一个bitmap作为背景。
![place holder](http://121.42.160.4:8081/memoryleak/EntranceFragment_image_exception3.png)

这是一个控件误用问题。BezelImageView通过画布裁切实现圆角效果，画布会一直保持mCacheBitmap的引用，不可为大图imageView使用。


# 2.SplashActiviy内存泄露优化

SplashActivity的内存泄露是很严重的问题，会一直持有闪屏图引用。同样是在类似页面有发现，同时LeakCancary也有提醒。  
![place holder](http://121.42.160.4:8081/memoryleak/SplashActivity_LeakCancary.jpg)
![place holder](http://121.42.160.4:8081/memoryleak/SplashActivity_Leak.png)

![place holder](http://121.42.160.4:8081/memoryleak/SplashActivity_Leak2.png)

问题分析：广告的数据库,AdDbHelper是个单例,一直持有Activity,没有释放。
最新的release SDk已经修复该问题。


# 3.首页优化运营图大小

运营图在mat中的bitmap在这里。




发现一个奇怪的问题，第一次网络拉取运营图，RGB_8888,尺寸2242928
第二次显示本地图，RGB_565,尺寸1121464。

出现该问题的原因是gradle的一个特性。Prefer_Value 并不是真实value.


基于此做了一些测试，以下是小图显示效果。

具体配置见上篇文章，传送门。

# 4.详情页内存泄露优化


非常简单的方法。来回切换10次，内存稳步增加，基本可以确定是泄露了

以BtsOrderDetailForDriverActivity内存泄露为例。每个实例占用150K空间


# 5.IMMessageActivity引导图优化。


# 6.IMMessageActivity内存泄露优化。



# 7.附MAT使用说明

此工具只能显示java层面的，而不能显示Ｃ层的内存占用信息。








[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/