---
layout: mypost
title: Android-自定义View基础
categories: [Android]
---
### 1.颜色
#### 1.1颜色基础
安卓支持的颜色模式：
|<font color=#87CEEB>颜色模式</font>  |<font color=#87CEEB>注释</font> | 
| ------------- |:-------------| 
| ARGB8888 | 四通道高精度(32位) | 
| ARGB4444 | 四通道低精度(16位) |
| RGB565| 屏幕默认模式(16位) |
| Alpha8	| 仅有透明通道(8位) |

以ARGB8888为例介绍颜色定义:
	
其中 A R G B 的取值范围均为0~255(即16进制的0x00~0xff)

A 从ox00到oxff表示从透明到不透明。

RGB 从0x00到0xff表示颜色从浅到深。

**当RGB全取最小值(0或0x000000)时颜色为黑色，全取最大值(255或0xffffff)时颜色为白色**
#### 1.2颜色使用
代码声明
```
int color = Color.GRAY;     //灰色

int color = Color.argb(127, 255, 0, 0);   //半透明红色

int color = 0xaaff0000;                   //带有透明度的红色
```
xml声明使用
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="color1">#f00</color> <!-- 低精度 - 不带透明通道红色 --> 
    <color name="color2">#af00 </color><!--低精度 - 带透明通道红色 -->
    <color name="color3">#ff0000</color><!--高精度 - 不带透明通道红色 -->
    <color name="color4">#aaff0000</color><!--高精度 -带透明通道红色 -->
</resources>
```
```
int color = getResources().getColor(R.color.color3);
```
在layout或style中使用

```
<!--在style文件中引用-->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorPrimary">@color/red</item>
</style>
```
```
android:background="@color/red"     //引用在/res/values/color.xml 中定义的颜色

android:background="#ff0000"        //创建并使用颜色
```

### 2.角度与弧度
在我们自定义View，尤其是制作一些复杂炫酷的效果的时候，实际上是将一些简单的东西通过数学上精密的计算组合到一起形成的效果。

|<font color=#87CEEB>名称</font>  |<font color=#87CEEB>定义</font> | 
| ------------- |:-------------| 
| 角度 | 两条射线从圆心向圆周射出，形成一个夹角和夹角正对的一段弧。当这段弧长正好等于圆周长的360分之一时，两条射线的夹角的大小为1度. | 
| 弧度| 两条射线从圆心向圆周射出，形成一个夹角和夹角正对的一段弧。当这段弧长正好等于圆的半径时，两条射线的夹角大小为1弧度. |
如图:
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/82568ecc7dd581cddd3987526944e6df.png)![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/508506a8e413abdd2caddb8456b9ef78.png)

圆一周对应的角度为360度(角度)，对应的弧度为2π弧度。

故得等价关系:360(角度) = 2π(弧度) ==> 180(角度) = π(弧度)

### 3.坐标系
 与数学中常见的坐标系不同 ，安卓设备屏幕左上角为坐标原点，向右为x轴增大方向，向下为y轴增大方向
如图：
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/2a5495c558128fe71b8b3b2a7eccc8a3.png)	![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/d28ee7f70a9a0236e5ab75e9bf8999f9.png)
PS: 同样的∠a 方向也与常见的Y轴相反。

View对应坐标系函数
```
getTop();       //获取子View左上角距父View顶部的距离
getLeft();      //获取子View左上角距父View左侧的距离
getBottom();    //获取子View右下角距父View顶部的距离
getRight();     //获取子View右下角距父View左侧的距离
```
```
event.getX();       //触摸点相对于其所在组件坐标系的坐标
event.getY();

event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
event.getRawY();
```

### 4.绘制流程
自定义View绘制流程函数调用链：
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/29b820ceda7b38683203f9c2a1e95cbc.png)

#### 4.1构造函数

```
public void MyView(Context context) {}
public void MyView(Context context, AttributeSet attrs) {}
public void MyView(Context context, AttributeSet attrs, int defStyleAttr) {}
public void MyView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}			//API21加入
```

一般常用的为第一个和第二个

```
//一般在直接New一个View的时候调用。
public void MyView(Context context) {}

//一般在layout文件中使用的时候会调用，关于它的所有属性(包括自定义属性)都会包含在attrs中传递进来。
public void MyView(Context context, AttributeSet attrs) {}
```
以下方法调用的是一个参数的构造函数：
```
//在Avtivity中
MyView view = new MyView(this);
```
以下方法调用的是两个参数的构造函数：
```
<!--在layout文件中 - 格式为： 包名.View名-->
<com.sloop.study.MyView
  android:layout_width"wrap_content"
  android:layout_height"wrap_content"/>
```

#### 4.2.测量View大小(onMeasure)

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthsize  MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
    int widthmode  MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式
    
    int heightsize  MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
    int heightmode  MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
}
```

onMeasure 函数中有 widthMeasureSpec 和 heightMeasureSpec 这两个 int 类型的参数， 它们不是宽和高， 而是由宽、高和各自方向上对应的测量模式来合成的一个值：
<font color=#CD9B1D>测量模式一共有三种， 被定义在 Android 中的 View 类的一个内部类View.MeasureSpec中：</font> 
|<font color=#CD9B1D>模式</font>  |<font color=#CD9B1D>二进制数值</font> | <font color=#CD9B1D>描述</font> | 
| ------------- |:-------------:| |-------------:| 
| UNSPECIFIED| 00 | 默认值，父控件没有给子view任何限制，子View可以设置为任意大小。|
| ARGB4444 | 01 |表示父控件已经确切的指定了子View的大小。|
| RGB565| 10 |表示子View具体大小没有尺寸限制，但是存在上限，上限一般为父View大小。|

**PS : 在实际运用之中只需要记住有三种模式，用 MeasureSpec 的 getSize是获取数值， getMode是获取模式。**

**如果对View的宽高进行修改了，不要调用 super.onMeasure( widthMeasureSpec, heightMeasureSpec) 要调用 setMeasuredDimension( widthsize, heightsize) 这个函数。**

#### 4.3确定子View布局位置(onLayout)
此方法分配大小和位置给它的每一个子View。

因此，我们在自定义一个扁平的自定义视图（继承简单的View）不具有任何子View，此时不需要重写此方法。而在自定义ViewGroup中会用到，他调用的是子View的layout函数。

在自定义ViewGroup中，onLayout一般是循环取出子View，然后经过计算得出各个子View位置的坐标值，然后用以下函数设置子View位置。

```
child.layout(l, t, r, b);
//l,t,r,b分别代表view距父view的左、顶、右、底的距离
```

#### 4.5.绘制内容(onDraw)

onDraw是实际绘制的部分，在这里，使用Canvas和Paint对象你将可以画任何你需要的东西。

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
}
```

一个Canvas实例从onDraw参数得来，它一般用于绘制不同形状，而Paint对象定义形状颜色。简单地说，Canvas用于绘制对象，而Paint用于造型。

#### 4.6.对外提供操作方法和监听回调

自定义完View之后，一般会对外暴露一些接口，用于控制View的状态等，或者监听View的变化.