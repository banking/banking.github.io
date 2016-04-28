---
layout: post
title: Android测量页面启动时间
date: 2016-04-22 11:12:05 +0800
categories: Android Optimize
---
# 1.Activity启动时间

Activity启动时会有onCreate/onStart/onResume这些回调，片面的在onResume回调时就认为Activity启动完成是不准确的。因为这时并没有真正执行View.onInflate().对于用户体验而言，应该归结到View第一帧绘制完成。参考[这里](https://www.zhihu.com/question/35487841)。   
以下的统计方法，都是以View绘制完成作为启动时间统计标准的。   
实际中，建议统计结果为5次的平均值。   


**1.ActivityManager Log.**    
Android系统的ActivityManager提供了Activity页面启动时间参数日志。    
统计方法:在debug日志中，筛选包含"ActivityManager: Displayed"字符串的日志。   
页面理想参数:350ms.   
该日志统计的是Activity从启动到页面绘制完成时间,关于ActivityManager统计时间细节，参考[这里](http://stackoverflow.com/questions/32844566/what-does-i-activitymanager-displayed-activity-850ms-comprised-of)。

**2.adb shell am**    
adb shell start是一项adb常用命令，可以外部调起方式调起Activity,Service,Broadcast等组件。   
需要注意，Activity必须要在Minifest注册时加export = "true"才可以；该属性管理Activity的外部调起权限。  
命令示例：adb shell am start -W -n "com.sdu.app.debug/com.sdu.app.MainActivity"    
输出示例：   
![Alt text](/assets/ActivityManager.png)   
 -W: wait for launch to complete 等待加载完成并输出日志    
参数说明：优先使用waittime作为结果（需要Android 5.0以上）；未输出waittime使用totaltime作为启动Activity的耗时统计结果。   
适合统计应用程序启动时间的场景。如果应用有一些广告页，可以绕过广告页直接加载MainActivity，得到真实的启动耗时参数。

**3.adb shell screenrecord**   
通过录屏方式分析启动时间。
http://www.jianshu.com/p/b3502562e55e   
http://graphics-geek.blogspot.sg/2015/10/measuring-activity-startup-time.html?m=1

命令示例。
1.开始录屏
adb shell screenrecord --bugreport /sdcard/video.mp4

2.停止录屏 control+c.

3.adb pull /sdcard/video.mp4 ~/Desktop/video.mp4


# 2.Fragment启动时间

**1.adb shell am**   
fragment在开发中也是广泛使用，有时有需求需要统计fragment的启动时间。   
在OnCreate中埋点作为启动开始时间。

    @Override
    public void onCreate(Bundle savedInstanceState) {
    	long startTime = System.currentTimeMillis();   
        super.onCreate(savedInstanceState);
    }
    
在View.post中埋点作为启动结束时间。

    public View onCreateView(...) {
        rootView = inflater.inflate(getLayoutId(), container, false);
        rootView.post(new Runnable() {
            @Override
            public void run() {
				long endTime = System.currentTimeMillis();
            }
        });
    }      
    
Android是消息驱动的模式，View.post的Runnable任务会被加入任务   队列，并且等待第一次TraversalRunnable执行结束后才执行。
ViewRootImpl会调用两次performTraversals，View.post中代码在第一次performTraversals后执行。
    
    


   





