---
layout: mypost
title: robotium在使用过程中的一些问题和知识点
categories: [自动化]
---
## 1.robotium在黑盒和白盒中的使用:
⑴ 黑盒测试需预先安装完重签名后的app,而白盒不用

⑵.白盒的测试程序初始化:


```
public XXX() {
           super (xxx.class) ;
}
```

黑盒的测试程序初始化:



```
private static Class<?> launchActivityClass;
static {// 由于没有源码，只能通过反射主类名，获取实例。
try {
launchActivityClass = Class. forName( xxx); 
} 
catch (ClassNotFoundException e) { 
throw new RuntimeException(e); 
}
}
public XXX() {
// 构造函数，传递包名和类名，供测试框架进行监控其状态，好对其进行模拟操作
super (packageName, launchActivityClass );
}
```





## 2.重签名
重签名可以用命令行和re-sign,如果使用的是re-sign.jar,re-sign使用的签名默认是C:\Users\Administrator\.android\debug.keystore,且暂时没办法更改,如果你已改动SDK的路径，导致原路径下的debug.keystore缺失,re-sign仍会显示重签名成功，但是实际只是去掉了签名，需手动命令行签名。
在用命令行使用debug.keystore进行重签名时,debug.keystore 签名密码为android，默认别名应为androiddebugkey，“找不到 xx.keystore证书链”为别名错误，可以用keytool.exe -list -keystore debug.keystore 获取别名。

## 3.robotium获取相同id或无id的的控件
很多控件都可以通过指定控件在界面上的排列顺序来定位。
通过index直接获取是最好的办法，但是当界面布局发生变化时，需及时维护和更新。
通过index获取指定控件的父控件，在通过getchildat获取指定控件，或者先获取子控件，再通过getparent获取父控件，此两类方法存在一定的问题。