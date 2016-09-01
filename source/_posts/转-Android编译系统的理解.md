---
title: '[转]Android编译系统的理解'
date: 2016-08-20 18:45:52
tags: Android
---
Android编译系统庞大，想要理解还是要花点功夫的，今天看到篇不错的帖子，特此转载。
### 简介
Android Build 系统是用来编译 Android 系统，Android SDK 以及相关文档的一套框架。众所周知，Android 是一个开源的操作系统。Android 的源码中包含了许许多多的模块。 不同产商的不同设备对于 Android 系统的定制都是不一样的。如何将这些模块统一管理起来，如何能够在不同的操作系统上进行编译，如何在编译时能够支持面向不同的硬件设备，不同的编译类型，且还要提供面向各个产商的定制扩展，是非常有难度的。 但 Android Build 系统很好的解决了这些问题，这里面有很多值得我们开发人员学习的地方。对于 Android 平台开发人员来说，本文可以帮助你熟悉你每天接触到的构建环境。对于其他开发人员来说，本文可以作为一个 GNU Make的使用案例，学习这些成功案例，可以提升我们的开发经验。
### 概述
Build 系统中最主要的处理逻辑都在 Make 文件中，而其他的脚本文件只是起到一些辅助作用，由于篇幅所限，本文只探讨 Make 文件中的内容。
    整个 Build 系统中的 Make 文件可以分为三类：
    第一类是 Build 系统核心文件
    此类文件定义了整个 Build 系统的框架，而其他所有 Make 文件都是在这个框架的基础上编写出来的。

图 1 是 Android 源码树的目录结构，Build 系统核心文件全部位于 /build/core（本文所提到的所有路径都是以 Android 源码树作为背景的，“/”指的是源码树的根目录，与文件系统无关）目录下。

![](www.ibm.com/developerworks/cn/opensource/os-cn-android-build/image001.png)

第二类是针对某个产品（一个产品可能是某个型号的手机或者平板电脑）的 Make 文件
这些文件通常位于 device 目录下，该目录下又以公司名以及产品名分为两级目录，图 2 是 device 目录下子目录的结构。对于一个产品的定义通常需要一组文件，这些文件共同构成了对于这个产品的定义。例如，/device/sony/it26 目录下的文件共同构成了对于 Sony LT26 型号手机的定义
![](http://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/image002.png)
第三类是针对某个模块（关于模块后文会详细讨论）的 Make 文件
 整个系统中，包含了大量的模块，每个模块都有一个专门的 Make 文件，这类文件的名称统一为“Android.mk”，该文件中定义了如何编译当前模块。Build 系统会在整个源码树中扫描名称为“Android.mk”的文件并根据其中的内容执行模块的编译。
### 编译Android系统
#### 执行编译
   Android 系统的编译环境目前只支持 Ubuntu 以及 Mac OS 两种操作系统。关于编译环境的构建方法请参见以下路径：http://source.android.com/source/initializing.html
在完成编译环境的准备工作以及获取到完整的 Android 源码之后，想要编译出整个 Android 系统非常的容易：
打开控制台之后转到 Android 源码的根目录，然后执行如清单 1

所示的三条命令即可（"$"是命令提示符，不是命令的一部分。）：

完整的编译时间依赖于编译主机的配置，在笔者的 Macbook Pro（OS X 10.8.2, i7 2G CPU，8G RAM, 120G SSD）上使用 8 个 Job 同时编译共需要一个半小时左右的时间。
清单一：编译Android系统
```
 $ source build/envsetup.sh 
 $ lunch full-eng 
 $ make -j8 
```
这三行命令的说明如下：

第一行命令“source build/envsetup.sh”引入了 build/envsetup.sh脚本。该脚本的作用是初始化编译环境，并引入一些辅助的 Shell 函数，这其中就包括第二步使用 lunch 函数。

除此之外，该文件中还定义了其他一些常用的函数，它们如表 1 所示：
表一：build/envsetup.sh中定义的常用函数
| 名称        | 说明         | 
| ------------- |-------------| 
|croot |	切换到源码树的根目录|
|m| 	在源码树的根目录执行 make|
|mm 	|Build 当前目录下的模块|
|mmm |	Build 指定目录下的模块|
|cgrep |	在所有 C/C++ 文件上执行 grep|
|jgrep |	在所有 Java 文件上执行 grep|
|resgrep |	在所有 res/*.xml 文件上执行 grep|
|godir 	|转到包含某个文件的目录路径|
|printconfig| 	显示当前 Build 的配置信息|
|add_lunch_combo |	在 lunch 函数的菜单中添加一个条目|

第二行命令“lunch full-eng”是调用 lunch 函数，并指定参数为“full-eng”。lunch 函数的参数用来指定此次编译的目标设备以及编译类型。在这里，这两个值分别是“full”和“eng”。“full”是 Android 源码中已经定义好的一种产品，是为模拟器而设置的。而编译类型会影响最终系统中包含的模块，关于编译类型将在表 7 中详细讲解。

如果调用 lunch 函数的时候没有指定参数，那么该函数将输出列表以供选择，该列表类似图 3 中的内容（列表的内容会根据当前 Build 系统中包含的产品配置而不同，具体参见后文“添加新的产品”），此时可以通过输入编号或者名称进行选择。
图三，lunch函数的输出
![](http://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/image003.png)
图 3. lunch 函数的输出
    第三行命令“make -j8”才真正开始执行编译。make 的参数“-j”指定了同时编译的 Job 数量，这是个整数，该值通常是编译主机 CPU 支持的并发线程总数的 1 倍或 2 倍（例如：在一个 4 核，每个核支持两个线程的 CPU 上，可以使用 make -j8 或 make -j16）。在调用 make 命令时，如果没有指定任何目标，则将使用默认的名称为“droid”目标，该目标会编译出完整的 Android 系统镜像。

### Build结果的目录结构

所有的编译产物都将位于 /out 目录下，该目录下主要有以下几个子目录：
    /out/host/：该目录下包含了针对主机的 Android 开发工具的产物。即 SDK 中的各种工具，例如：emulator，adb，aapt 等。
    /out/target/common/：该目录下包含了针对设备的共通的编译产物，主要是 Java 应用代码和 Java 库。
    /out/target/product/<product_name>/：包含了针对特定设备的编译结果以及平台相关的 C/C++ 库和二进制文件。其中，<product_name>是具体目标设备的名称。
    /out/dist/：包含了为多种分发而准备的包，通过“make disttarget”将文件拷贝到该目录，默认的编译目标不会产生该目录。

Build生成的镜像文件：

Build 的产物中最重要的是三个镜像文件，它们都位于 /out/target/product/<product_name>/ 目录下。

这三个文件是：
    system.img：包含了 Android OS 的系统文件，库，可执行文件以及预置的应用程序，将被挂载为根分区。
    ramdisk.img：在启动时将被 Linux 内核挂载为只读分区，它包含了 /init 文件和一些配置文件。它用来挂载其他系统镜像并启动 init 进程。
    userdata.img：将被挂载为 /data，包含了应用程序相关的数据以及和用户相关的数据。
    
原文较长，这里只转载了一部分。
原文地址：http://blog.csdn.net/huangyabin001/article/details/36383031





