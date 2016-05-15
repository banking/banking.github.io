---
layout: post
title: 通过Mat利器OQL分析Android App内存
date: 2016-05-11 10:41:17 +0800
categories: Android Optimize
---
内存分析离不开Mat,在Mat上接触到了OQL.
另：OQL对宏观上了解App组件的和JVM虚拟机的学习，也大有益处。


# 1.什么是OQL，如何使用

在Mat中使用OQL.



# 2.OQL语法简介
**OQL支持的关键字**
OQL是类似SQL的一种语法。



**OQL语法特性**
INSTANCEOF关键字:查找属于某父类，实现某接口对象。

@usedHeapSize：对象属性，占用内存值
@retainedHeapSize：对象属性，包含引用对象占用内存值
@GCRootInfo：对象的垃圾回收根节点，判断是否被引用


支持正则："java\.lang\..*"
**使用技巧**
常见Mat中使用，关心目标对象的占用内存(usedHeapSize)，引用对象占用内存（retainedHeapSize）。推荐在Select语句中标记显示这些属性。
下面是一条OQL：   
SELECT s AS Value, s.@usedHeapSize AS "Shallow Size", s.@retainedHeapSize AS "Retained Size" FROM INSTANCEOF android.content.BroadcastReceiver s 

# 3.十分实用的OQL语句

1.查找所有Activity:    
作用：过滤出所有Activity实例。检查Activity泄露   
语句： SELECT * FROM INSTANCEOF android.support.v4.app.FragmentActivity    
说明：android.support.v4.app.FragmentActivity 可以替换为android.app.Activity；多个相同Activity实例可能存在泄露



2.查找所有Fragment:   
作用：查看Fragment泄露,如ViewPager多次切换可能泄露Fragment   实例   
语句：SELECT f AS Value, f.@usedHeapSize AS "Shallow Size", f.@retainedHeapSize AS "Retained Size" FROM INSTANCEOF android.support.v4.app.Fragment f  
说明：多个相同fragment实例可能存在泄露，同Activity

3.SELECT file.path.toString() FROM java.io.File file
作用：查看文件目录

4.查看所有字符串   
SELECT toString(s) AS Value, s.@usedHeapSize AS "Shallow Size", s.@retainedHeapSize AS "Retained Size" FROM java.lang.String s 



#4.OQL Report

参考文献
更多官方语法：
http://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Freference%2Foqlsyntax.html