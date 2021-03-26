---
layout: mypost
title: HTTPS执行流程解析
categories: [网络]
---
>你是否还在组件化开发中使用他人打好的aar包，而每次因为代码的更新都要更换arr包而感到不厌其烦。其实，在AS种简单配置一下Maven就可以解决这个困扰。
###Maven是什么？
Maven是一个项目管理工具，它包含了一个项目对象模型 (POM：Project Object Model)，一组标准集合，一个项目生命周期(Project Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段中插件目标的逻辑。
![黑人问号.png](https://upload-images.jianshu.io/upload_images/8557875-08652de9fab20e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####首先列举几个开发中经常要做的事情
- 我们需要引用各种jar包，尤其是比较大的工程，引用的jar包往往有几十个乃至上百个， 每用到一种jar包，都需要手动引入工程目录，而且经常遇到各种让人抓狂的jar包冲突，版本冲突。

- 我们辛辛苦苦写好了Java文件，可是只懂0和1的白痴电脑却完全读不懂，需要将它编译成二进制字节码。好歹现在这项工作可以由各种集成开发工具帮我们完成，Eclipse、IDEA等都可以将代码即时编译。当然，如果你嫌生命漫长，何不铺张，也可以用记事本来敲代码，然后用javac命令一个个地去编译，逗电脑玩。

- 世界上没有不存在bug的代码，正如世界上没有不喜欢美女的男人一样。写完了代码，我们还要写一些单元测试，然后一个个的运行来检验代码质量。

- 再优雅的代码也是要出来卖的。我们后面还需要把代码与各种配置文件、资源整合到一起，定型打包，如果是web项目，还需要将之发布到服务器，供人蹂躏。

Maven是这样一种工具，它可以把你从上面的繁琐工作中解放出来，能帮你构建工程，管理jar包，编译代码，还能帮你自动运行单元测试，打包，生成报表，甚至能帮你部署项目，生成Web站点。
#####Android Studio中Maven仓库的使用
- Maven有一个用处-依赖管理，通过在pom中指定坐标的形式将jar引入到项目中。
- 以上过程经历怎样的流程？从哪里寻找jar包？下载的jar放在哪里?
- Maven通过仓库来统一管理各种构件。
![image.png](https://upload-images.jianshu.io/upload_images/8557875-24a44285c975585d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/8557875-bda54839123a5bcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####搭建远程maven项目
  PS：略过本地Maven项目发布（实际意义不大）

1.  获取到Nexus的远程仓库地址，格式一般为：
      http://192.168.aa.bbb:port/nexus/content/repositories/releases/
      http://192.168.aa.bbb:port/nexus/content/repositories/snapshots/
2. 配置AS
      1)重新配置gradle.properties
      2)在library moudle新建maven_push.gradle
![image.png](https://upload-images.jianshu.io/upload_images/8557875-2326a62e024d3dd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/8557875-f0fe2e60339c0672.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/8557875-dfa08ceab0f92df8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  3)在library的build.gradle文件头添加
```
apply from: 'maven_push.gradle'
```

 4)Gradle任务窗口点击运行“uploadArchives”

 5)打开nexus查看上传文件
![图片2.png](https://upload-images.jianshu.io/upload_images/8557875-8b33762310dac625.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    6)引用时将本地仓库路径更换为远程仓库路径：
```
maven{
    url 'http://192.168.aa.bbb:port/nexus/content/repositories/releases/'  
}
```
```
compile GROUP:POM_ARTIFACT_ID:VERSION
```
到此，我们引用的组件更新后，只要更改VERSION再编译后就是最新的组件代码了。
