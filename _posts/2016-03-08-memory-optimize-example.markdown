---
layout: post
title:  "Android 内存优化案例"
date:   2016-03-07 19:57:04 +0800
categories: jekyll update
---

整理一下顺风车4.2.4版本中的内存优化案例。  

# 1.首页优化运营图背景

第一次通过Mat分析观察到了异常Bitmap对象,猜测和首页运营图相关。
![place holder](http://121.42.160.4:8081/memoryleak/EntranceFragment_image_exception1.png)

下面可以看到已经定位到了业务代码BtsHomeImageView。
![place holder](http://121.42.160.4:8081/memoryleak/EntranceFragment_image_exception2.png)

继续跟进这个控件，可以看到其父类BezelImageView在初始化时创建了一个bitmap作为背景。
![place holder](http://121.42.160.4:8081/memoryleak/EntranceFragment_image_exception3.png)

这是一个控件误用问题。BezelImageView通过画布裁切实现圆角效果，画布会一直保持mCacheBitmap的引用，不可为大图imageView使用。


# 2.SplashActiviy内存泄露优化

SplashActivity的内存泄露是很严重的问题，会一直持有闪屏图引用。同样是在类似页面有发现，同时LeakCancary也有提醒。  
![place holder](http://121.42.160.4:8081/memoryleak/SplashActivity_LeakCancary.jpg)
仍然是使用Mat追踪问题：
![place holder](http://121.42.160.4:8081/memoryleak/SplashActivity_Leak.png)
![place holder](http://121.42.160.4:8081/memoryleak/SplashActivity_Leak2.png)

问题分析：广告的数据库,AdDbHelper是个单例,一直持有Activity,没有释放。
最新的theone-SDk 4.3.27已经修复该问题。


# 3.首页优化运营图大小

运营图在mat中的bitmap在这里。
![place holder](http://121.42.160.4:8081/memoryleak/Home_Operation_bitmap.png)

发现一个奇怪的问题，第一次网络拉取运营图，RGB_8888,大小2242928byte.
第二次显示本地图，RGB_565,大小1121464byte。
回溯了一下glide源码，出现该问题的原因是gilde的一个特性。代码如下：
![place holder](http://121.42.160.4:8081/memoryleak/glide_prefer.png)

后面有想过针对图片大小做优化，基于此做了一些测试，以下是小图显示效果，对比如下:
![place holder](http://121.42.160.4:8081/memoryleak/Operation_scale.png)

具体配置标准见[上篇文章](http://banking.github.io/jekyll/update/2016/03/07/android-memory-optimize.html)。

# 4.详情页内存泄露优化

对于首页，详情页这样负责的页面，检查内存泄露有更简单的方法。
多次进入目标页面并返回，内存稳步增加，基本可以确定是泄露了。
初始内存：
![place holder](http://121.42.160.4:8081/memoryleak/Memory_check.png)

进入页面十几次后内存：
![place holder](http://121.42.160.4:8081/memoryleak/Memory_check2.png)

以BtsOrderDetailForDriverActivity内存泄露为例,Dump内存后可以看到内存中一直存在多个实例。
![place holder](http://121.42.160.4:8081/memoryleak/Memory_check3.jpg)

每个实例占用150K空间。
![place holder](http://121.42.160.4:8081/memoryleak/Memory_check4.jpg)

如果内存泄露暂时无法处理。保底方案：在Activity onDestory时候从view的rootview开始，递归释放所有子view涉及的图片，背景，DrawingCache，监听器等等资源，让Activity成为一个不占资源的空壳，泄露了也不会导致图片资源被持有。
详文在[这里](https://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=400656149&idx=1&sn=122b4f4965fafebf78ec0b4fce2ef62a&scene=1&srcid=0304Y3NE2XzzpXnciVmy8V3p&key=710a5d99946419d991561737ff21afe36087e30f7048e503c104b3e7f52184650009e924294d02ce511f14198c71a882&ascene=0&uin=MjQxMDYzNTU%3D&devicetype=iMac+MacBookAir5%2C2+OSX+OSX+10.11.3+build(15D21)&version=11000003&pass_ticket=C1SGUVOM%2FbWwXCSLYZ6k4PrmyBNZxI5EE66XuGZcRjk%3D)。

# 5.IMMessageActivity引导图优化。
进入IM页面OOM崩溃问题是上版本的顽疾。初次进入发现激增20M内存，必然存在严重内存问题。
分析发现有异常大图出现。
![place holder](http://121.42.160.4:8081/memoryleak/IMMessageActivity_Guide_Exception1.png)
发现是一个引导图对象，属于初始化加载错误。
![place holder](http://121.42.160.4:8081/memoryleak/IMMessageActivity_Guide_Exception2.png)

# 6.IMMessageActivity内存泄露优化。
IMMessageActivity页面存在多处内存泄露。引发原因多是（匿名）内部类持有activity引用没有释放引起。
比如这个：
![place holder](http://121.42.160.4:8081/memoryleak/IMMessageActivity_Leak.png)
在Activity的OnDestroy时及时解除引用就可以了。














[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/