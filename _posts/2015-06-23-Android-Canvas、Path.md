---
layout: mypost
title: Android-Canvas、Path
categories: [Android]
---
### **1.Canvas**
Canvas我们可以称为画布，能够在上面绘制各种东西，是图形绘制的基础。

特点:
1.可操作性强：由于这些是构成上层的基础，所以可操作性必然十分强大。
2.比较难用：各种方法太过基础，想要完美的将这些操作组合起来有一定难度。

#### **Canvas的常用操作速查表**

|<font color=#CD9B1D>操作类型| <font color=#CD9B1D>相关API|<font color=#CD9B1D> 备注|
| ------------- |:-------------| -----|
| 绘制颜色 | drawColor, drawRGB, drawARGB | 使用单一颜色填充整个画布 |
| 绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc| 	依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧 |
| 绘制图片| drawBitmap, drawPicture| 	绘制位图和图片 |
| 绘制文本| drawText, drawPosText, drawTextOnPath| 	依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字|
| 绘制路径| drawPath| 绘制路径，绘制贝塞尔曲线时也需要用到该函数|
| 顶点操作| drawVertices, drawBitmapMesh| 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用|
| 画布剪裁| clipPath, clipRect| 设置画布的显示区域|
| 画布快照| save, restore, saveLayerXxx, restoreToCount, getSaveCount| 	依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数|
| 画布变换| translate, scale, rotate, skew| 依次为 位移、缩放、 旋转、错切|
| Matrix(矩阵)| getMatrix, setMatrix, concat| 实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。|
参考资料：[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)	

### **2.Path**

Path含义，  官方介绍：
The Path class encapsulates compound (multiple contour) geometric paths consisting of straight line segments, quadratic curves, and cubic curves. It can be drawn with canvas.drawPath(path, paint), either filled or stroked (based on the paint’s Style), or it can be used for clipping or to draw text on a path.

简单解释就是：
Path封装了由直线和曲线(二次，三次贝塞尔曲线)构成的几何路径。你能用Canvas中的drawPath来把这条路径画出来(同样支持Paint的不同绘制模式)，也可以用于剪裁画布和根据路径绘制文字。我们有时会用Path来描述一个图像的轮廓，所以也会称为轮廓线(轮廓线仅是Path的一种使用方法，两者并不等价)

在使用Canvas时绘制的都是简单图形(如 矩形 圆 圆弧等)，而对于那些复杂一点的图形则没法去绘制(如绘制一个心形 正多边形 五角星等)，而使用Path不仅能够绘制简单图形，也可以绘制这些比较复杂的图形。另外，根据路径绘制文本和剪裁画布都会用到Path。


#### **Path常用方法表**

|<font color=#CD9B1D>操作类型| <font color=#CD9B1D>相关API|<font color=#CD9B1D> 备注|
| ------------- |:-------------| -----|
| 连接直线 | lineTo | 添加上一个点到当前点之间的直线到Path |
|闭合路径 |close| 连接第一个点连接到最后一个点，形成一个闭合区域 |
|添加内容| addRect, addRoundRect, addOval, addCircle, addPath, addArc, arcTo| 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)|
|是否为空|isEmpty| 		判断Path是否为空|
|是否为矩形|isRect| 	判断path是否是一个矩形|
| 贝塞尔曲线|	quadTo, cubicTo|分别为二次和三次贝塞尔曲线的方法|
|rXxx方法|rMoveTo, rLineTo, rQuadTo, rCubicTo| 	不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)|
参考资料：[Path](http://developer.android.com/reference/android/graphics/Path.html)
	
		

		
		
		
	
	


		
		