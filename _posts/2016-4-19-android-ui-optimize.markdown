---
layout: post
title:  "Android 流畅性优化"
date:   2016-04-19 13:55:04 +0800
categories: Android Optimize
---
# 1.过度绘制优化
Android对过度绘制有很方便的检测手段。(设置->开发者选项->调试GPU过度绘制->显示过度绘制区域/显示过度绘制计数器)   
**过度绘制是什么** ：绘画了多次页面元素。
“显示过度绘制计数器”会统计某页面UI绘制的平均值，根据我个人的开发经验，2层左右的绘制是最优值，要优化超过3层的过度绘制。   
**检查多余背景** ：多余背景是最容易出现过度绘制的原因。   
**Theme引发过度绘制** 
目前顺风车页面层级是这样的。  
 
很明显绘制了两次背景，有两种方法优化：1.把Theme中的background去掉,具体页面使用特定背景色；2.使用Theme中的background作为统一背景色，页面根部局不再设置背景色。
目前考虑到页面差异（如详情和列表背景色不一致），选择了第一种方式，修改方法是在Application和Activity的主题中都添加这行代码：
   
&lt;item name="android:windowBackground"&gt;@null&lt;/item&gt;






# 2.优化布局层次
对布局参数最准确是测量工具是Hierarchy Viewer.   
- **include标签** 
最重要的还是重用布局。对优化布局层次无作用  
**merge标签** ：在可以使用时尽量使用。
自定义布局，尽量少使用padding。这样可以使用merge标签     

**LinearLayout/RelativieLayout**
一层的RelativeLayout效率高于二层LinearLayout.
layout_weight属性的LinearLayout会在绘制时花费昂贵的系统资源，因为每一个子组件都需要被测量两次
 





# 3.减少不必要的view Inflate。  
**ViewStub**  
不得不提到ViewStub.它提供了一种方便的LazyLoad方式处理View Inflate.回到业务场景，非常适合小概率需要显示的View做LazyLoad。
 
ViewStub vs View.Gone?
我们在实际使用中，有很多布局是少概率出现的,比如网络异常布局，新手引导布局等。最合适的方式是使用ViewStub。 二者的效率差别参考
[这篇文章](http://magicmicky.github.io/android_development/benchmark-using-traceview/)的第二个试验，作者测量了ViewStub和View.Gone消耗Cpu时间，大家有兴趣也自己试验一下。作者结论如下:   
Inflating a more complex layout, even when some views’ visibility are set to GONE, takes more time than inflating the simpler view, without the hidden views.     
 
 




# 4.TraceView分析优化  
其他影响建议使用TraceView分析。TraceView是Android支持的一种，实时测量某时间区间cpu时间片函数调用情况。
可能导致Ui卡顿的常见场景:数据解析，数据库操作，网络等。
比如，Volley框架中Request putParam是在主线程的。如果参数过多，或者有加密操作，可能会消耗主线程过多时间。


 



[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/