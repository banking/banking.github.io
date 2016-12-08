---
layout: post
title: 通过Mat利器OQL分析Android App内存
date: 2016-05-11 10:41:17 +0800
categories: Android Optimize
---

内存分析离不开Mat,我也正是在Mat使用过程中接触到了OQL，发现OQL是内存分析的一大利器。   
下面总结一下OQL相关的知识。


# 1.什么是OQL，如何使用

OQL的全称是Object Query Language，是一门用类SQL语言查找内存镜像中目标对象的语言，是一种领域专属语言（DSL）。

在Mat中使用OQL：
需要打来Mat软件，并导入需要分析的hprof文件。当文件被Mat解析完成后，就能在Mat导航栏里看到「OQL」的图标，点击就能输入并执行OQL了。     

>优点：分析详细，根据你的筛选需求详尽的列出所有内存对象。   
>缺点：需要一次次dump heap分析，时间成本很高。


# 2.OQL语法简介
**OQL支持的关键字**
OQL语法非常类似SQL，但由于分析目标都为内存中对象，所以可以直接引用对象的多种属性。

**OQL语法特性**   

- INSTANCEOF 关键字:查找属于某父类，实现某接口对象。
- @usedHeapSize：对象属性，获取占用内存值    
- @retainedHeapSize：对象属性，获取包含引用对象占用总内存值   
- @GCRootInfo：对象属性，表示改对象的垃圾回收根节点，判断是否仍被引用   
- 支持正则：例如"java\.lang\..*"   

**使用技巧**
常见Mat中使用，关心目标对象的占用内存(usedHeapSize)，引用对象占用内存（retainedHeapSize）。推荐在Select语句中标记显示这些属性。    
下面是一条OQL：      
SELECT s AS Value, s.@usedHeapSize AS "Shallow Size", s.@retainedHeapSize AS "Retained Size" FROM INSTANCEOF android.content.BroadcastReceiver s 

# 3.Android中通用的OQL语句

##1.查找所有Activity:    
作用：过滤出所有Activity实例。检查Activity泄露   
语句： SELECT * FROM INSTANCEOF android.support.v4.app.FragmentActivity    
说明：android.support.v4.app.FragmentActivity 可以替换为android.app.Activity；多个相同Activity实例可能存在泄露



##2.查找所有Fragment:   
作用：查看Fragment泄露,如ViewPager多次切换可能泄露Fragment   实例   
语句：SELECT f AS Value, f.@usedHeapSize AS "Shallow Size", f.@retainedHeapSize AS "Retained Size" FROM INSTANCEOF android.support.v4.app.Fragment f  
说明：多个相同fragment实例可能存在泄露，同Activity

##3.查看文件目录
SELECT file.path.toString() FROM java.io.File file
说明：可以查到内存中图片，字节流等和文件关联

##4.查看所有字符串   
SELECT toString(s) AS Value, s.@usedHeapSize AS "Shallow Size", s.@retainedHeapSize AS "Retained Size" FROM java.lang.String s 

# 4.业务使用的OQL语句
#1.查找所有自定义Widget:  
比如说我现在项目的自定义Widget，都放在了com.didi.theonebts.widget包中，可以这样写OQL：       
SELECT widget AS Value, widget.@usedHeapSize AS "Shallow Size", widget.@retainedHeapSize AS "Retained Size" FROM INSTANCEOF "com.didi.theonebts.widget.*" widget     
#2.查找所有的classloader
SELECT classof(cl) AS VALUE FROM INSTANCEOF java.lang.ClassLoader cl    
查到所有的classloader  


参考文献    
更多官方语法：      
http://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Freference%2Foqlsyntax.html