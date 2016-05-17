---
layout: post
title:  "Android 流畅性优化"
date:   2016-04-19 13:55:04 +0800
categories: Android Optimize
---
页面流畅性是App非常重要的性能指标，下面总结一下流畅性优化容易出效果的点。
# 1.过度绘制优化
 
**过度绘制检查** ：过度绘制指屏幕上的某个像素在同一帧的时间内被绘制了多次。
多层次重叠的UI结构里面，如果不可见的UI也在做绘制的操作，会导致某些像素区域被绘制了多次，从而会浪费大量的CPU以及GPU资源。   
过度绘制是页面流畅的大敌，Android对过度绘制有方便的检测手段：设置->开发者选项->调试GPU过度绘制->显示过度绘制区域/显示过度绘制计数器。   
“显示过度绘制计数器”会统计某页面UI绘制的平均值，根据我个人的开发经验，2层以内的绘制数是最优值，一定要优化超过3层的过度绘制。   
**检查多余背景** ：多余背景是最容易出现过度绘制的原因。通过细致的代码review可以发现很明显的多余背景；建议通过官方HierarchyViewer检查。      
在成功Load View Hierarchy后，在DecorView中执行Profile Node，就可以在布局节点中看到每个view的UI显示情况；我们可以从DecorView一直跟踪到页面中具体Widget的背景，排查并去掉多余背景。

   
**优化Theme过度绘制** 
举例来说，某页面层级是这样的： 
![place holder](http://121.42.160.4:8081/overdraw/android_overdraw.png) 

DecorView绘制了主题背景，RelativeLayout@25ec597作为Activity根布局也绘制了背景。很明显这里有OverDraw，两种方法可以优化：   
1.把Theme中的background去掉,具体页面使用特定背景色；   
2.使用Theme中的background作为统一背景色，页面根部局不再设置背景色。   
目前考虑到页面差异（如详情和列表背景色不一致），选择了第一种方式，修改方法是在Application和Activity的主题中都添加这行代码：
   
&lt;item name="android:windowBackground"&gt;@null&lt;/item&gt;      
巧合的是，在Android Developer的官方blog中也提到了这种优化方式。<sub><font  color="0099ff" face="黑体">[1]</font></sub>






# 2.优化布局层次
同上，对布局层级最准确的检测工具是Hierarchy Viewer.   
**include标签** 
确切说是用来实现重用布局，做到NRY。对优化布局层次无作用。
**merge标签** ：
在可以使用时尽量使用，使用场景：
（1）父布局只有一个子布局，且该子布局和父布局是同一种布局时。
（2）根布局是frameLayout时，可以将其替换成merge标签。
（3）自定义布局时，尽量不使用padding；这样可以使用merge标签，减少一层布局。   

**LinearLayout/RelativieLayout**
一层的RelativeLayout效率高于两层LinearLayout，所以复杂的布局多使用RelativeLayout。   
如果相同层次的LinearLayout可以实现RelativeLayout的布局效果，显然使用LinearLayout。
layout_weight属性的LinearLayout会在绘制时花费昂贵的系统资源，因为每一个子组件都需要被测量两次；请尽量少使用。
 





# 3.减少不必要的view Inflate  
**ViewStub**  这里不得不提到ViewStub的使用。它提供了一种方便的LazyLoad方式，来处理Activity OnCreate中大量View Inflate造成的卡顿问题。   
回到业务场景，非常适合小概率需要显示的View使用：比如网络异常View,新手引导图等。    
用户操作触发的View显示：如edittext获取焦点后弹出的表情输入View。

 
**ViewStub vs Gone** 我们在实际使用中，有很多布局是少概率出现的,比如网络异常布局，新手引导布局等。最合适的方式是使用ViewStub。 二者的效率差别在
参考文献2<sub><font  color="0099ff" face="黑体">[2]</font></sub>的第二个试验，作者测量了ViewStub和View.setVisibility(Gone)消耗Cpu时间，ViewStub的执行效率明显优于Gone.   
作者结论如下:   
Inflating a more complex layout, even when some views’ visibility are set to GONE, takes more time than inflating the simpler view, without the hidden views.   
 
# 4.DecorView.post()
[上篇文章](http://banking.github.io/android/optimize/2016/04/22/android-test-page-launch-time.html)提到过，可以根据DecorView.post来测量页面启动时间。
同样此方案非常适合做页面启动时的流畅性优化,比如在updateYourUI()中执行影响页面启动性能的操作。示例代码如下：     
 
	getWindow().getDecorView().post(new Runnable() {
	    @Override
	    public void run() {
	        myHandler.post(mLoadingRunnable);
	    }
	});    
	
	private Handler myHandler = new Handler();
	private Runnable mLoadingRunnable = new Runnable() {
	
	  @Override
	  public void run() {
	    updateYourUI(); 
	  }
	};




# 5.TraceView分析  
建议使用TraceView分析页面启动时间点的cpu消耗。TraceView可以准确测量某时间区间内cpu时间片各函数执行情况，关注非UI更新类型的cpu消耗。可能导致Ui卡顿的常见场景:数据解析，数据库操作，网络等。   
**优化例子** 下面是一个页面启动时的TraceView分析图：
![place holder](http://121.42.160.4:8081/overdraw/http_put_params_traceview.jpg)
原因分析：Volley框架中Request putParam是在主线程的。如果参数过多，或者有大段字符加密操作，会消耗主线程过多时间。    
优化方法：将大部分不变化的params做缓存，在内存中保留一个存放这些params的Map；在网络请求时直接执行putAll()操作。
 

参考文献：
1.http://android-developers.blogspot.jp/2009/03/window-backgrounds-ui-speed.html
2.http://magicmicky.github.io/android_development/benchmark-using-traceview/
