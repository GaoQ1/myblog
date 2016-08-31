---
title: CSS_3D_Panorama
date: 2016-08-31 10:37:01
tags:
- css
categories: 转载笔记
---
> Tomorrow is never clear.Our time is here.

## 前言
3D全景并不是什么新鲜事物了，但以前我们在Web上看到的3D全景一般是通过flash实现的。若我们能将CSS3 Transform的相关知识运用得当，也是能实现类似的效果。换句话说，3D全景其实就是CSS3 3D的应用场景之一。

## 准备
在实现CSS3 3D全景之前，我们先理清部分CSS3 Transform属性：
 - Transform-origin: 元素变形的原点（默认值为 50% 50% 0，该数值和后续提及的百分比均默认基于元素自身的宽高算出具体数值）;
 - perspective: 指定了观察者与z=0平面的距离，使具有三维位置变换的元素产生透视效果（默认值：none，值只能是绝对长度，即负数是非法值）;
 - transform-style: 用于指定其为子元素提供2D还是3D的场景。另外，该属性是非继承的。
 - transform: 该属性能让你修改CSS可视化模型的坐标空间，其中包括平移(translate)、旋转(rotate)、缩放(scale)和扭曲(skew).

下面对上述的一些点进行更深入的分析：
 - perspective，该属性指定了“眼睛”与元素的`perspective-origin`(默认值为50% 50%)点的距离。那么问题来了：“当我们应用px作为衡量单位时，那它的实际距离该如何量化呢（即如何得到我们熟悉且易于表达的距离）?”  当我们的屏幕的分辨率是1080p(1920*1080px)，且该元素或祖先元素的perspective值为1920px时，该应用了CSS3 3D Transform的元素的立体效果就相当于我们我们在一个屏幕宽度(1920px)的屏幕前观看该元素的真是效果。尽管如此，目前我也不清楚如何准确地为元素设置一个合适的perspective的值，只能猜测个大概值，然后再动态修改值，以达到满意的呈现效果。
![呈现效果](/images/CSS3D/perspective.jpg)
根据相似三角形的性质可计算出被前移的元素最终在屏幕上显示的实际大小

另外，关于perspective还有另外一个重要的点。因为，`perspective-origin`的默认值为50% 50%,因此对哪个元素应用perspective属性，就决定了“眼睛”的位置(即我们的“眼睛”是在哪个角度看物体)。一般来说，当我们需要正视物体时，就会将该属性设置在与 **该元素中心重合的某一祖先元素上**。

再另外，如果说“如何让一个元素(背面)不可见?”，你可能会说backface-visibility:hidden。其实，对于在“眼睛”背后的元素(以元素的`transform-origin`为参考点)， **即元素的z轴坐标值大于perspective的值，浏览器是不会将其渲染出来的**。

 - 对于`transform-style`，该属性指定了其子元素是处于3D场景还是2D场景。对于2D场景，元素的前后位置是按照平时的渲染方式(即若在普通文档流中，是按照代码中元素的先后顺序，后面的元素会遮住在其前面的元素)；对于3D场景，元素的前后位置是按照真实世界的规则排序(即越靠近“眼睛”的，会遮住离“眼睛”更远的元素)。 另外，由于 **transform-style属性是非继承性的，对于中间节点需要显示设定**
 - 对于transform属性：下图整理了rotate3d、translate3d的变化方向：
![变换方向](/images/CSS3D/transform.jpg)
transform中的变换属性的顺序是有关系的，如transformX(10px)rotate(30deg)与rotate(30deg)transformX(10px)是不等价的。

另外，需要注意的是scale中如果有负数值，则该方向会产生180度的翻转。再另外，部分transform效果会导致元素(字体)模糊，如translate的数值存在小数、通过translateZ或scale方法元素等等。每个浏览器都有其不同的表现。

## 实现
上面理清了一些CSS Transform的知识点，下面就讲讲如何实现CSS 3D全景：

想象一下，当我们站在十字路口中间，身体不动，头部旋转 360°，这个过程中所看到的画面就形成了一幅以你为中心的全景图了。其实，当焦距不变时，我们就等同于站在一个圆柱体的中心。

但是，虚拟世界与现实的最大不同是：没有东西是连续的，即所有东西都是离散的。例如，你无法在屏幕上显示一个完美的圆。你只能以一个正多边形表示圆：边越多，圆就越“完美”。

同理，在三维空间，每个 3D 模型都等同于一个多面体（即 3D 模型只能由不弯曲的平面组成）。当我们讨论一个本身就是多面体（如立方体）的模型时并不足以为奇，但当我们想展示其它模型时，如球体时，就需要记住这个原理了。
![球体模型](/images/CSS3D/ball.png)

[淘宝造物节的活动页](http://zwj360.im20.com.cn/)就是CSS 3D全景的一个很赞的页面，它将全景图分隔成20等份，相邻的元素间差18deg(360/20)。需要注意的是：我们要确保每个元素的正面是指向圆柱中心的，所以要计算好每等份的旋转角度值。然后再将元素向外(即Z轴方向)平移r px.对于立方体r就是边长/2，而对于其他更复杂的正多面体呢？

举例：对于正九面体，每个元素的宽为210px，对应的角度为40deg，例如下图：
![正九面体的俯视图](/images/CSS3D/diagram.png)
![计算过程](/images/CSS3D/calc.png)

由此得到一个公用函数，只需传入含有元素的宽度和元素数量的对象，即可得到r值：
```javascript
  function calTranslateZ(opts){
    return Math.round(opts.width/(2*Math.tan(Math.PI/opts.number)))
  }

  calTranslateZ({
    width: 210,
    number: 9
  })
```
![俯视时所看到的元素外移动画](/images/CSS3D/expand4.gif)

另外，为了让下文易于理解，我们约定html结构是：
```javascript
  #view(perspective:1000px)
    #stage(transform-style: perserve-3d)
      #cube(transform-style: perserve-3d)
        .div(width:600px;height:600px) /*组成立方体的元素*/
```

正多面体构建完成后，就需要将我们的“眼睛”放置在正多面体内。由于在“眼睛”后的元素是不会被浏览器所渲染的（与 .div元素 是否设置 backface-visibility:hidden; 无关），且我们保证了.div元素的正面都指向了正多面体的中心，这就形成360deg被环绕的效果了。
那“眼睛”具体是被放置在哪个位置？
答：通过设置#stage元素的translateZ的值，让不能被看到的.div元素的Z轴坐标值大于perspective的值即可。如：立方体的正面的translateZ是-300px(为了保证立方体的正面是指向立方体中心，正面元素需要设置rotate(-180deg) translateZ(-300px), 即正面元素向“眼球”方向平移了300px),而#view的perspective的值为1000px，那么 #stage 的 translateZ 的值应该大于 700px 且小于 1300px，具体数值则取决于你想要呈现的效果了。

可由下图看出，将水平的 4 张图片合成后就是一张全景图：
![4张图合成的全景图](/images/CSS3D/panorama3d.jpg)

理解上述知识后，就可以通过为元素设置合适的 CSS3 Transform 属性值，即可实现一张可交互的全景图了。当然，交互的效果可以是拖拽，也可以是重力感应等。

正如在上文提到的：“没有东西是连续的，即所有东西都是离散的…”。将《造物节》与后续全景图的水平方向上的图片分别合成一张图后，可以发现：图片数量越多，图片的要求也越低。你觉得呢？
![造物节全景图](/images/CSS3D/zaowujie.jpg)

## 全景图素材的制作
将全景图制作分为设计类与实景类：

### 设计类
要制作类似 [《淘宝造物节》](http://zwj360.im20.com.cn/) 的全景页面，设计稿需要有以下这些要求。

整体背景设计图如下（2580*1170px，被分成 20 等份）：
![造物节全景图](/images/CSS3D/zaowujie.jpg)

基本要求：
 - 水平方向上需要首尾相连；
 - 因为效果图最终需要切成 N 等份，所以尽可能让 设计图的宽度能被 N 整除；
 - 图片尺寸不仅要考虑正视图的大小，还要考虑元素在旋转时依然能覆盖视野（可选）。

当然，上图只是背景图，还可以添加一些小物体素材（通过运动速度的差异形成视差，增强立体效果），如：
![小物体元素1](/images/CSS3D/item3.jpg)
![小物体元素2](/images/CSS3D/item1.jpg)

小物体元素（虚线是用于参考的，造物节中共有 21 个小物体）

如上图所示，每个图片也是被等分成 M 等份。当然，M 取决于物体在背景上的具体位置和本身大小。
另外，M 的宽度是与 N 的宽度相等的。尽管部分物体（M>1）的两侧等份的图案占比小，但建议保留同样的宽度。

注：如果小物体有特殊的变形效果，应该备注具体变形参数。

对于顶部和底图图片，则无特殊要求。

### 实景类
如果想制作实景的全景，可以看看 Google 街景：
[Google街景](https://www.google.com/streetview/publish/)推荐的设备如下：
![实景类工具](/images/CSS3D/panorama_machine.jpg)

处理全景的方式：
 - 利用 RICOH THETA S 等专业设备拍出全景图
 - 导出静态图像
 - 利用设备专门提供的 APP 或 krpamo tools、pano2vr、Glsky box 等工具将静态图像转为6张图
 - 利用 Web 技术制作可交互的全景图

其中 Web 技术有以下3种可选方式（当然，还有其它）：
 - CSS3（本文所提及的方式）
 - Three.js
 - krpano（为全景而生，低级浏览器则回退到 Flash），[查看教程](http://krpano.com/docu/tutorials/quickstart/?from=groupmessage&isappinstalled=0#top)

可见，优秀硬件设备的出现，大大减少了后期处理的时间，而 Web 则提供了一个很好的展现平台。

## 最后
随着终端设备的软硬件不断完善和提高，Web 在 3D 领域也不甘落后，如果你玩腻了 2D 的 H5 或者想为用户提供更加新颖优秀的体验，全景也许是一种选择。
