---
layout: post
title: "Android Studio - gradle配置踩坑"
date: 2016-12-8 23:00
comments: true
tags: 
	- Android Studio
	- 配置
---
##### 1.问题
当我们从团队协作者或者开源Android项目网站上获取以Gradle构建的Android项目，并点击[Open an existing Android Studio project](https://developer.android.com/studio/intro/migrate.html)将它导入Android Studio时，经常遇到如下Android Studio未响应的问题：

![Building XXX Gradle project info](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/Building_XXX_Gradle_project_info.png)

##### 2.分析
查看idea.log，我们得知问题是由`Started sync with Gradle`时造成`Connection Failed`而引发的。
![idea.log](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/idea_log.png)

原因之一可能是该Android项目所需的Gradle版本与本地存在的Gradle版本并不适配。因此，Android Studio会自动去下载该Android项目所需的Gradle版本。
> 然而，却跪倒在GFW面前。

##### 3.解决
通过在Android Studio中依次点击`File > Settings > Build, Execution, Deployment > Gradle`,我们可以锁定当前项目使用的Gradle的位置。[2]

	- 若选中`Use default gradle wrapper(recommended)`,则设置的Gradle位置为`Service directory path`中的路径；
	- 若选中`Use local gradle distribution`，则设置的Gradle位置为`Gradle home`中的路径。
> 注：`Service directory path`是全局级的，`Use default gradle wrapper(recommended)`与`Use local gradle distribution`是项目级的，优先级高于全局级的设置。

![Gradle Location](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/Gradle_Location.png)

因此，根据当前设置，我们在`C:\Users\%Your_User_Name%\.gradle\wrapper\dists`目录下找到当前已安装的Gradle，
![.gradle/wrapper/dists](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/gradle_dists.png)
并确保Gradle的完整性。
![ensure gradle's integrity](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/gradle_integrity.png)

下图是Android Studio Gradle插件版本与Gradle版本之间的对应关系[1]。
> Gradle插件版本一般继承自Android Studio的版本。

![plugin version|required gradle version](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/gradle_version.png)

博主的本地环境为

| plugin version | required gradle version |
| -------------- | ----------------------- |
| 2.2.2          | 2.14.1                  |

因此，对该Android项目的`build.gradle`和`gradle\wrapper\gradle-wrapper.properties`这两个文件进行修改[1]。
![gradle-wrapper.properties](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/gradle_wrapper.png)

![build.gradle](http://oi0xi3dzx.bkt.clouddn.com/web/android/basic/build.gradle.png)

修改过后，可对该Android项目进行正常导入，并且不会出现上述的未响应情况。


参考文献：
[1 Android Plugin for Gradle Release Notes](https://developer.android.com/studio/releases/gradle-plugin.html)
[2 Configure Gradle project settings](https://www.jetbrains.com/help/idea/2016.2/gradle-2.html)