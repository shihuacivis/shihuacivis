title: 用CSS3画一个奥迪车标
date: 2015-10-30
---

又要到周末啦，今天下班路上突然脑洞大开，作为一个有『汽车情怀』的前端，竟然没有做过汽车相关专题的玩意，于是就想到了做一个车标LOGO。

说到车标，当然是要先做我心爱的奥迪啦！

<!-- more -->

下面是实现思路：

首先是HTML结构： 

```HTML
<div class="logo">AUDI</div>
```

然后是CSS：

```CSS
.logo {
	color: red;
}
```

**大功告成！so easy！**


**纳尼？我裤子都脱了你就让我看这个？？？**

开个玩笑。。。


本文将涉及下面几个知识点： 

> 1. CSS3画圆形、圆环
> 2. 通过transform实现水平和垂直居中
> 3. rem的响应式解决方案
> 4. CSS3渐变滤镜


下面进入正题。

先一睹为快: [点我看线上demo](http://demo.qpdiy.com/sh/vehicleLogo/)

首先讲大致的思路：

奥迪嘛，四个圈咯。

那么要做的就是画4个圆环， 想做得逼真一点，就要再给圆环加上渐变阴影了。

ok，那么解决的步奏就是：

> 1. 画圆环
> 2. 给圆环加渐变阴影
> 3. 四个圆环定位布局

思路清晰之后就开始了。


## 画圆环

CSS3还没有强大到自动无脑画圆环，所以智慧的劳动人民们一般是这样实现的：

### 实现方法
假设要画一个灰色的圆环，那么：
1. 画一个大的灰色的圆；
2. 再画一个和背景同色的圆，居中盖在灰色的圆环上。
那么这样看上去就是一个中空的灰色圆环啦。

### 如何画圆
这里有一个基础的技巧： 

将一个 正方形div（width和height相等）的 圆角属性设为 50% 即可 获得一个直径 = width 的圆

```CSS
.cirle {
	width: 100px;
	height: 100px;
	border-radius: 50%;
}
```

### 开始画我心爱的奥迪环了

首先我们要模仿的LOGO大概如下面这张图：

![cmd-markdown-logo](/img/2015091101.jpg)

每个奥迪环的最外层的表面可以看做一个环，然后内侧也是一个环，所以每个奥迪环里其实要画两个环。
所以HTML结构可以考虑设置如下：

```HTML
  <div class="logo-circle-wrap l1">
    <div class="circle-fir">
      <div class="circle-sec">
        <div class="circle-thi"></div>
      </div>
    </div>
  </div>
```
要画两个圆环，所以每个奥迪环要配置三个圆形： 

第一层是家族式外壳， 第二层是大众爹的动力总成，第三层是自我标榜的内饰……

圆环基本结构画好后，开始在渐变上作文章了。

CSS3中提供了线性渐变滤镜可以用于div的背景中，以后再详细分析这一属性，感兴趣的可以自己查一下。

```CSS
	background-image: linear-gradient(0deg, #000000 .5%, transparent 35%);
```

下面就是整个圆面实现的css了，我这里用了rem的自适应方案，不太清楚rem的同学可以自行了解一下。。。
```CSS
.logo-circle-wrap {
    width: 5rem;
    height: 5rem;
    position: absolute;
}
.circle-fir {
  display: block;
  position: relative;
  width: 5rem;
  height: 5rem;
  border-radius: 50%;
  border: .05rem solid #6f6f6f;
  background-color: #ebebeb;
  background-image: linear-gradient(15deg, #000000 .5%, transparent 35%);
}

.circle-fir .circle-sec {
  color: #232323;
  position: absolute;
  left: 50%;
  top: 50%;
  width: 4rem;
  height: 4rem;
  border-radius: 50%;
  border: .1rem solid #efefef;
  background-image: linear-gradient(33deg, #888888 10%, #454545 66.7%, transparent);
  -webkit-transform: translate(-50%, -50%);
          transform: translate(-50%, -50%);
}

.circle-fir .circle-thi {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 3.5rem;
  height: 3.5rem;
  background-color: #fff;
  border-radius: 50%;
  overflow: hidden;
  -webkit-transform: translate(-50%, -50%);
          transform: translate(-50%, -50%);
}
```

画完了一个圆，其它的就都依样画葫芦啦。
将四个圆位置调整一下

```CSS
.logo-circle-wrap.l1 {
  left: 0rem; 
}
.logo-circle-wrap.l2 {
  left: 3rem; 
}
.logo-circle-wrap.l3 {
  left: 6rem; 
}
.logo-circle-wrap.l4 {
  left: 9rem; 
}
```
### div覆盖解决方案

这时候就发现一个蛋疼的问题了，后面的奥迪环会将前一个奥迪环的右半部分遮住。
这时候第一反映就是要死要死要死了，因为这些奥迪环本来就是由三层结构叠加起来了，想做圆环内部背景透明似乎不可行。

这时候奥迪的好友『丰田』上线了，正所谓『车到山前必有路，有路必有丰田车』（广告费请洽门卫李大爷）

既然背景不透明，就在后面的环上再画上前一个环的右半部分。这样看起来就像是透明的了。

然而问题又来了，怎样只画一部分圆环呢？

你可能想到，在后面的环里再嵌入一个大环，然后把一部分遮住就可以了。

我确实是这么做的，再内层环里设置overflow:hidden，然后嵌入的环通过transform属性拉到和前一个环同一位置。

```HTML
<div class="logo-circle-wrap l2">
  <div class="circle-fir">
    <div class="circle-sec">
      <div class="circle-thi">
        <!-- 第一个环的右半部分 -->
        <div class="logo-circle-wrap part part1">
          <div class="circle-fir">
            <div class="circle-sec">
              <div class="circle-thi"></div>
            </div>
          </div>
        </div>
        <!-- 第一个环的右半部分 end -->
      </div>
    </div>
  </div>
</div>
```

```CSS

.circle-fir .circle-thi {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 3.5rem;
  height: 3.5rem;
  background-color: #fff;
  border-radius: 50%;
  overflow: hidden;
  -webkit-transform: translate(-50%, -50%);
          transform: translate(-50%, -50%);
}

.logo-circle-wrap.part{
    width: 5rem;
    height: 5rem;
    position: absolute;
    overflow: hidden;
    -webkit-transform: translate(-3.75rem, -.7rem);
            transform: translate(-3.75rem, -.7rem);
}

```


大功告成！

线上的DEMO地址:[点我看线上demo](http://shihuacivis.github.io/css3audi/)

