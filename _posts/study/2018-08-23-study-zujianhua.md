---
layout:     post
title:      "组件化"
subtitle:   "Upgrading Ele.me to Progressive Web App"
date:       2018-08-23 12:00:00
author:     "Hale"
header-mask: 0.3
catalog:    true
tags:
    - Android
---


> 组件化是将系统分解为可以轻松重组的独立可重用包的过程。


## 定义
组件化是将系统分解为可以轻松重组的独立可重用包的过程。 


## 模块化,组件化,插件化区别

[引用1][1]和[引用2][2]  
1. 模块化就是将一个程序按照其功能做拆分，是一个总的概念  
2. 组件化是建立在模块化基础上，强调编译时可拆分，每个组件既可以是一个library又可以是完整的Application  
3. 插件化是建立在模块化基础上，强调运行时可拆分，每个插件都是完整的应用  

![123](https://upload-images.jianshu.io/upload_images/6650461-c64f921882f379ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700/format/webp)

## Android组件化实施

[滴滴的组件化实践与优化](http://www.infoq.com/cn/articles/xiaojukeji-component-practice-and-optimization)

[Android组件化方案](https://blog.csdn.net/guiying712/article/details/55213884)

## Android组件化开源库
[ARouter](https://github.com/alibaba/ARouter)
阿里的一个用于帮助 Android App 进行组件化改造的框架 —— 支持模块间的路由、通信、解耦

## 技巧  
1. 可以考虑把主module作为app壳，不写任何业务逻辑，代码都在子module中  


2.  library和Application转换，Gradle构建的工程中，module是由apply plugin:'com.android.application'来标识Application,apply plugin: 'com.android.library'来标识Library.因此,我们可以在编译时通过判断构建环境中的参数来修改子工程的工作方式,让debug时作为Application,release时为Library
如在module的gradle脚本头部加入以下脚本片段:  

```
if (isDebug.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
```

```
// 作为library时不能有applicationId,只有作为一个独立应用时才能够如下设置
if (isDebug.toBoolean()){
    applicationId "com.wuxiaolong.module1"
}
```
```
//指定不同的清单文件，区分是否需要Application 和 launch 的Activity
sourceSets {
        main {
            if (isDebug.toBoolean()) {
                manifest.srcFile 'src/main/module/AndroidManifest.xml'
            } else {
                manifest.srcFile 'src/main/AndroidManifest.xml'
            }
        }
    }
```
  

3. 当module数量过多时会影响编译速度，可以考虑把不经常改动的模块打包成AAR远程依赖。
  

4. 当分别开发模块时，容易出资源重复命名的问题，可以通过给模块设置不同的资源前缀，避免重复命名。在build.gradle中设置  
```
resourcePrefix "module1_"
```
resourcePrefix 这个值只能限定 xml 里面的资源，并不能限定图片资源，所有图片资源仍然需要你手动去修改资源名或者也可以单独有一个供各组件共用的resouse组件专门存放共用资源

## DEMO
[https://github.com/guiying712/AndroidModulePattern/](https://github.com/guiying712/AndroidModulePattern/)  

[https://github.com/WuXiaolong/ModularSample](https://github.com/WuXiaolong/ModularSample)



[1]: https://blog.csdn.net/fepengwang/article/details/80533301
[2]: https://blog.csdn.net/dd864140130/article/details/53645290
