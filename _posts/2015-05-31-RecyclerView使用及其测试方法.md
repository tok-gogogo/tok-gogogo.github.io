---
layout: mypost
title: RecyclerView使用及其测试方法
categories: [Android]
---
# RecyclerView

RecyclerView自android5.0出来已经1年多了，我们看一下那些还在坚持listview等人的想法：


- **ListView能添加头部，RecyclerView不能**
- **ListView稳定，还有很多开源库**

-------------------

## RecyclerView与Listview的比较
RecyclerView控件是比ListView更先进、灵活的版本，是ListView的升级版。
为什么这么说：

 - RecyclerView最大的优势就是灵活，RecyclerView只需改变一行代码就可以变化多种不同的布局显示排版

 - RecyclerView.Adapter，比BaseAdapter做了更好的封装，把BaseAdapter的getView方法拆分成onCreateViewHolder方法和onBindViewHolder方法，强制需要创建ViewHolder，这样的好处就是避免了初学者写性能不佳的代码
 

## 使用ReclyclerView 

 - 添加分割线
 - 列表动画
 - 改变某个数据保持当前位置
 - 添加头部尾部
 - 列表分组

####添加分割线

```java
mRecyclerView.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.VERTICAL_LIST));
```
####列表动画
推荐一个RecyclerView的动画库（[recyclerview-animators](https://github.com/wasabeef/recyclerview-animators)）

RecyclerView自带添加、删除动画，而ListView则需添加额外的代码才能实现。
删除调用RecyclerView的adapter的notifyItemRemoved
添加调用RecyclerView的adapter的notifyItemInserted
我们可以发现RecyclerView.Adapter和BaseAdapter相比，额外提供了一下这些方法：
```java
// 数据发生了改变，那调用这个方法，传入改变对象的位置。
public final void notifyItemChanged(int position);
// 可以刷新从positionStart开始itemCount数量的item了
public final void notifyItemRangeChanged(int positionStart, int itemCount);
// 添加，传入对象的位置。
public final void notifyItemInserted(int position);
// 删除，传入对象的位置。
public final void notifyItemRemoved(int position);
// 对象从fromPosition移动到toPosition 
public final void notifyItemMoved(int fromPosition, int toPosition); 
//批量添加 
public final void notifyItemRangeInserted(int positionStart, int itemCount);
//批量删除
public final void notifyItemRangeRemoved(int positionStart, int itemCount);
```
####改变某个数据保持当前位置

RecyclerView改变了以往Listview改变列表某一个item数据，然后刷新列表，刷新后则会回到最顶部，而同样的操作但是原来滑动的位置不变。

####添加头部尾部
推荐一个RecyclerViewHeader（[RecyclerViewHeader](https://github.com/blipinsk/RecyclerViewHeader)）
两种使用RecyclerViewHeader的方法：

1.为header创建一个xml布局（可以包括任意view或者ViewGroup）
```java
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="100dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="header"/>

</FrameLayout>
```
```java
RecyclerViewHeader header = RecyclerViewHeader.fromXml(context, R.layout.header);
```
将RecyclerViewHeader Attach 到RecyclerView，搞定！
```java
RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
// set LayoutManager for your RecyclerView
header.attachTo(recyclerView);
Header-already-aligned approach (不会引入任何额外布局):
```
2.将RecyclerViewHeader布局放在RecyclerView的上层。
```java
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal|top" />

    <com.bartoszlipinski.recyclerviewheader.RecyclerViewHeader
        android:id="@+id/header"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_gravity="center_horizontal|top">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="header"/>

    </com.bartoszlipinski.recyclerviewheader.RecyclerViewHeader>

</FrameLayout>
```
获得RecyclerViewHeader对象：
```java
RecyclerViewHeader header = (RecyclerViewHeader) findViewById(R.id.header);
把RecyclerViewHeader赋予RecyclerView

RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
// set LayoutManager for your RecyclerView
header.attachTo(recyclerView, true);
```
注意事项
RecyclerViewHeader必须在RecyclerView设置了LayoutManager之后调用。
目前该库适用于LinearLayoutManager,GridLayoutManager和StaggeredGridLayoutManager布局的RecyclerViews。
只支持垂直布局LayoutManager

####列表分组
[IndexRecyclerViewht](tps://github.com/jiang111/IndexRecyclerView)
通过RecyclerView实现分组功能的联系人的效果。
1.首字母悬浮在顶部。
2.侧滑删除联系人。
3.联系人索引。

##RecyclerView在自动化测试中的应用

5.5.4版本robotium更新后
[robotium版本更新](http://blog.csdn.net/gongke425281007/article/details/51007310)

solo类提供了以下对recyclerview的操作
```java
clickInRecyclerView(int itemIndex) 
clickInRecyclerView(int itemIndex, int recyclerViewIndex) 
clickLongInRecycleView(int itemIndex) 
clickLongInRecycleView(int itemIndex, int recyclerViewIndex) 
clickLongInRecycleView(int itemIndex, int recyclerViewIndex, int time) 
scrollDownRecyclerView(int index) 
scrollRecyclerViewToBottom(int index)
scrollUpRecyclerView(int index)
scrollRecyclerViewToTop(int index)

```