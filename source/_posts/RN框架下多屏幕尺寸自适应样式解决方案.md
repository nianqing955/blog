---
title: RN框架下多屏幕尺寸自适应样式解决方案
date: 2017-03-10 15:22:27
tags: react native
---
### RN框架下多屏幕尺寸自适应样式解决方案

@(react native)[ 屏幕适配|]

[TOC]

#### 引言
 >屏幕适配是一个很容易被忽略的问题，但对于精益求精的产品而言，是必不可少的。移动端的适配的目的是让UI效果在不同分辨率的设备下，样式不会走形。换言之，就是如何同一套代码在不同分辨率的手机上跑时，页面元素间的布局间距，留白，以及图片大小会随着变化，在比例上跟设计稿一致。
在web中，可以通过多种方式来进行适配，比如设置viewpoint、media查询等，因此对于嵌入移动的H5页面的适配问题，本文将不做讨论和展开。React native是一套开源的框架，借助RN可以快速的开发出移动APP,包括IOS及Android两端。同样RN框架也存在着多屏幕尺寸的适配问题，并且对Android的支持不是很友好。

#### 问题分析
>不同的机型手机的屏幕尺寸不同，像素级别也不一样，所以样式的展示方式会存在差异。在RN的开发框架下，采用的是flex的布局方式，而flex的布局中，是不支持根据屏幕的尺寸，来进行百分比布局的，因此在设置宽度或者高度时是不需要单位的。
    在RN中的单位是pt，因此可以通过Dimensions 来获取设备的宽高，PixelRatio 获取设备屏幕密度值。

#### 解决方案
1.	flex弹性布局

采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称"项目"。
容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size，如图1所示。
![](/images/2-1.png)

   在移动端的设备中一般都是采用flex弹性布局的方式，在设置容器中的项目宽度时，可以根据flexDirection 来设置项目的排列方式，项目会根据flex的值来划分所占的宽高。
   
   示例：ios6 和ios6plus 下的布局自适应
 ![](/images/2-2.png)
 如上图2所示，在代码中我们按行进行排列，设置第一行的高度为deviceHight/2.容器的显示方式为flex。
至此，我认为在容器当中，不能设置定高，比如height：200，应当进行换算，比如deviceHight/n来设置。而容器中的项目，可以根据容器的高度来进行定高的划分。那么布局上就可以满足自适应的需求。

2. 图片自适应

在不同的屏幕下，所表现的图标清晰度及大小会发生改变，那么具体的解决方式有如下几种。
安装第三发的npm的包：
（1）react-native-fit-image 类似于image组件，示例如图3所示，ios6 plus和Ios7比较。
![](/images/2-3.png)
代码如图4所示：
![](/images/2-8.png)
 经过判断，该包可以满足图片的在设备上的自适应。
 （2）react-native-style-loader 可以使用web css的方式来进行RN适配的包。支持media查询、支持css中的px, vw, vh, rem, pt 单位。

3. 可以按官网中的现有的文档适配图片布局

（1） 官网中Image包含resizeMode这一属性，属性值包括contain, cover, stretch。默认不设置模式等于cover模式。如图5所示，三种模式的不同效果。
cover模式自适应宽高，给出高度值即可；
contain铺满容器，但是会做截取；
stretch铺满容器，拉伸；
![](/images/2-4.png)
目前我们的项目中，所用的模式是属于默认模式，也就是cover，就如图5中最上层样式。
（2）官网中还有一种方式，是通过PixelRatio 获取屏幕的像素密度，从而达到图片的自适应。如图6所示。
![](/images/2-5.png)
由图6中可以看出，像素密度的值只有3个：1，2，3，即可以通过设置图片的宽高＊像素密度值来加载不同的图片。目前项目中，默认也是通过这种方式来加载图片。
（3）可以根据不同设备的宽高，将 UI 等比放大到 app 上的自适应布局，假设需要100*100的图。代码如图7所示。
![](/images/2-6.png)
（4）可以根据设备的宽高比，来取得屏幕的像素点的值，最后进行样式转换。如图8所示。
![](/images/2-7.png)
#### 四、总结
    RN基于pt为单位，可以通过Dimensions来获取宽高，PixelRatio 获取密度，不支持百分比，但可以通过获取屏幕宽度手动计算进行百分比布局。图片的自适应方式上，可以按需求来规划，具体方案还需要讨论再确定。方案的撰写过程中，感谢同事天宝和鹏飞的帮助，给了一些解决方案。

#### 参考资料
（1）flex布局:http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
（2）react－native 布局: https://segmentfault.com/a/1190000002658374
（3）react-native-fit-image: https://github.com/huiseoul/react-native-fit-image
（4）react-native-style-loader:https://github.com/dengchengmi/react-native-style-loader
（5）Responsive Design: https://medium.com/@elieslama/responsive-design-in-react-native-876ea9cd72a8 - .hx2xn94fp
