---
layout: post
title:  "Android 内存优化方案和总结"
date:   2016-03-07 19:57:04 +0800
categories: jekyll update
---
# 1.Activity内存泄露检查
使用LeakCancary,Memory Dump,Mat分析。    
**Memory Dump**：检测手段最简单。在Package Tree View页面筛选，理论上，检测任何到activity个数>1都应该提出质疑。    
**LeakCancary**：配置方便，也比较准确。Launcher中打开Leaks就可以查看到内存泄露的Activity汇总。    
**Mat**：最全面，但稍显复杂。需要查看Histogram中异常Activity个数，观察手动gc后某些activity是否及时销毁，对业务代码的熟悉程度有较高要求。    
**解决方案** ：相关RD检查代码，是否有持有context引用未释放的情况，并修复之。禁止外部静态单例强引用context.  
**进度** ： 目前已经修复SplashActivity内存泄露，IM会话详情Activity内存泄露；乘客详情Activity，司机详情Activity由腾讯地图引起，正在推动解决。  




# 2.Bitmap优化异常 
- **异常大图对象** 
属于代码逻辑错误导致的，在打开某些页面时，进行Memory检测,如果内存有异常增长，可以确认问题存在。  
**解决方案** ：打开页面后立刻dump memory，并使用mat分析。在Dominator Tree定位异常bitmap对象引用，检查代码修复。  
**进度** ：顺风车首页解决了一个误用控件绘制bitmap背景的问题；IM详情页解决了一个过早初始化新手引导图的问题。  
- **现有图片瘦身** 
打开某些页面，dump memory heap，并用mat分析内存对象。中观察bitmap，定位到图片的代码归属 。建议运营图不要超过1M，全屏浮层图大小不要超过4M。
bitmap在内存中大小计算规则：长*宽*单像素内存大小。有alpha通道的RGB_8888单像素占内存4byte,无alpha通道的RGB_565单像素占内存2byte.  
**解决方案** ： 所有网络下发的大图采用无alpha通道的jpg格式，而不是png。服务器配置多图，客户端根据手机分辨率拉取合适的图。
比如顺风车首页运营图，配置两套图，分别为1066*526和720*356。UI同事建议，需要保持2/3以上的失真率。1920*1080尺寸的手机，可以选择720*356尺寸图，更大尺寸手机选择1066*526。  
**进度** ：目前顺风车首页运营图格式已经优化；图片大小配置需要配合API和UI同事进行。




# 3.代码层面优化。  
- **store优化**  
TheOne框架引中，引入了store管理activity逻辑的代码架构。提供了快速创建Sotre单例的方式:SingletonHolder.getInstance(XXX.class)。
但是SingletonHolder用了强引用的方式缓存了sore对象，导致store会一直存在于内存中。  
**解决方案** ：
1.如果store为单个activity服务，建议不采用单例方式，直接new来创建store对象  
2.如果store为多个activity服务，必须有clear方法，结合业务需求清除store内部的数据和对象。在某些业务逻辑触发后（比如发单成功后清理PublishStroe)。  

- **其他代码层面优化**  
减少静态单例使用。静态单例多在工具类中出现，视业务区分，不要过度使用静态单例，并注意适时回收。  
减少enum使用，使用SparseArray取代Map等。




# 4.跟踪Omega线上异常，在触发OOM 节点检查代码。  
触发oom肯定这时需要大量的内存分配，才会发生OOM错误。  
以顺风车为例。前三项OOM异常，分别是：     
（1）IM详情页OnCreate时触发。    
（2）订单详情页OnCreate时触发。     
（3） 网络解析response json时触发。    
内存优化是否成功的检查唯一标准，一定是线上OOM爆发量是否减少。所以在触发OOM的节点，一点也要做细致的code review。  
**进度**：目前在三处都检查到了一些问题，做以下优化。  
（1）IM详情页 发现了一个进入页面后所有gif表情，轮播多次的异常逻辑。已修复。  
（2）订单详情页 有内存泄露情况发生。  
（3）列表页pageNum = 30 导致回文过大（目前是160KB)，考虑优化pageNum = 15。  



[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/