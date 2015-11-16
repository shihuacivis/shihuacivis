title: 用cocos2d-js-lite版本制作一个小游戏——数钱大作战
---

cocos2d-js-lite是一个cocos2d的轻量化精简版本，只保留了cocos2d-js的一些核心方法。
它所包含的特性列表如下：
    + Canvas渲染器 (不支持WebGL)
    + 场景和图层
    + 事件管理器
    + 计时器
    + 精灵和贴图
    + TTF文本
    + 声音
    + 动作
    + 菜单和菜单项

与cocos2d-js完整版一个最大的不同就是，lite版把引擎的所有模块都放到一个文件里，这对于一个第一次接触游戏引擎的js开发者来说是一个福音，开发者只需像引入一个jQuery库一样引入一个文件即可。

**另外，lite是基于纯原生js的，所以无须像`白鹭引擎`（基于Typescript）一样要先经过编译才能再浏览器中打开，因此再整个工作流上感觉与传统前端开发更为接近**

从lite版包含的特性来看，它更适合开发轻度的H5游戏。

*首先要吐槽一下lite版本不支持的2个实用功能*
> 1. 不支持位图字体labelBMFont方法
> 2. 不支持从plist生成元素（当然传统前端并不知道这有啥用）


## 为何要用cocos2d-js-lite

首先，在不使用游戏引擎的前提下，也可以用纯原生js + canvas + div + css制作出一款H5游戏。
所以要明确用这个游戏引擎的目的。

cocos下面几个特性可以提升游戏的开发进度和质量。
> 1. 引擎提供了资源预加载的方法，可以加载多种资源
> 2. 引擎可以非常简单的制作各种位移、拉伸、旋转等动画
> 3. 引擎提供了事件的监听和管理
> 4. 引擎提供了强大无脑的屏幕自适应方案
> 5. 引擎提供了高性能的声音及音效的播放功能
> 6. 引擎本身文件仅300kb，对于H5应用可以接受

*好戏开场了*

*cocos2d的世界中的元素*

cocos2d中，主要包含下面四层元素：
> 1. view：可以看作是一个视窗，即游戏内容的载体（可以比作舞台）。
> 2. Scene：舞台中一级容器就是场景。
> 3. Layer：场景中又可以分成很多『层』。
> 4. Sprite：每一个层中又可以包含很多个小的元素(如舞台上的角色）。


跟HTML中的div盒模型做类比，以上元素的对应关系大致如下：
view —— document、window
Scene —— html
Layer —— body
Sprite —— div、p等



## 如何启动游戏引擎和开启游戏


### 1.如何启动引擎
lite版本中启动引擎的方法非常简单，只需像引入jquery一样在html中引入'cocos2d-js-v3.7-lite.js'文件即可。
然后在在body中加入一个canvas元素，然后在游戏配置文件（即根目录下的project.json）的`id`字段中填入该canvas的id即可。

```html
<script type="text/javascript" src="cocos2d-js-v3.7-lite.js" charset="UTF-8"></script>
<body>
<canvas id="gameCanvas" width="750" height="1334"></canvas>
</body>
```

### 2.如何开启游戏

首先看看游戏配置文件（即根目录下的project.json）：

```javascript
{
    "debugMode"     : 1,
    "frameRate"     : 60,
    "id"            : "gameCanvas",
    "renderMode"    : 1,
    "showFPS"       : true,
    "jsList"        : [
      "j/GameScene.js"
    , "j/StartScene.js"
    , "j/GameControler.js"
    ]
}
```
我们可以在该文件中设置调试模式、帧数、渲染模式、是否显示FPS信息等等。

当游戏引擎文件加载完毕后，就可以用下面方法启动游戏了。
```javascript
  cc.game.run();
```
它首先根游戏的配置文件进行初始化设置，然后加载配置文件的「jsList」中罗列的文件。
当这些js文件都加载完毕后，就会触发一个cc.game.onStart事件。
所以我们在调用cc.game.run方法前，先要定义cc.game.onStart事件：

```javascript
cc.game.onStart = function(){
  // 检测浏览器环境，如果是ios下则启动retina模式，有助于增强引擎渲染效果（可选项）
  if (cc.sys.IOS || cc.sys.OS_OSX) {
    cc.view.enableRetina(true);
  }
  // 设置游戏的预设大小（即设计稿大小）和适配方案
  cc.view.setDesignResolutionSize(750, 1334, cc.ResolutionPolicy.EXACT_FIT);
  // 让游戏随着浏览器屏幕大小伸缩变化
  cc.view.resizeWithBrowserSize(true);
  //加载静态资源，包括图片、声音、字体等资源，res_list是一个包含资源路径的数组
  cc.LoaderScene.preload(res_list, function () {  
      cc.director.runScene(new GameScene());
  }, this);
};
```

### 自适应方案

引擎提供了屏幕的五种适配方案，我们只需要设置好游戏预设的大小（即设计稿的大小）以及[适配的方案](http://http://www.cocos2d-x.org/docs/manual/framework/html5/v2/resolution-policy-design/zh), 引擎就会自动完成游戏适配工作。于此同时我们在游戏中对元素的布局可以完全根据设计稿来1：1的制作，而再也不用自己费心去做适配的计算和处理啦！
比如这个游戏的设计稿是iphone6的屏幕大小，然后我选用的适配方案是全拉伸到覆盖屏幕的全部区域
```javascript
cc.view.setDesignResolutionSize(750, 1334, cc.ResolutionPolicy.EXACT_FIT);
```


### 预加载
引擎自带了一个预加载页面，只需要调用cc.LoaderScene.preload即可。

**如果需要自定义加载页面可能需要修改引擎源码。或者通过cc.Loader.load方法自定义一个加载模块**

当资源加载完毕后，我们可以调用cc.director.runScene()进入某以个场景，如上面代码中的GameScene。

## 游戏场景 Scene

在开启一个场景(Scene)前,我们要先定义它，首先用cc.Scene.extend方法返回一个Scene对象，
这里的entend类似java等oop语言中的`继承`的概念……
简单的说就是通过这个方法，新生成的这个类就具备了跟`原始类`（cc.Scene）的属性和方法。同时我们可以对原始类中定义书属性和方法进行`重写`。
比如下面的代码中，我们对onEnter方法进行了重写。
onEnter是Scene对象中最重要的方法，当我们切换到某个场景时就会触发这个方法。
通常我们习惯在这个时候对Scene进行各种初始化（元素的布局定位和变量设置）。

但要注意这里不能省去调用this._super()方法，这个方法的含义是去调用父类的`构造函数`（即cc.Scene的构造函数），将GameScene类`实例化`，当我们把类实例化之后，才能愉快的调用（访问）它的各个方法和变量。

简单的说，在进入场景时，我们要先初始化场景，调用this._super(),然后再进行其它操作。 

```javascript
var GameScene = cc.Scene.extend({
  onEnter:function () {
    this._super();
}）
```

## 场景分层 Layer

Scene的下一层容器即为Layer，当然，Scene也可以直接插入Sprite元素。
**实际上cocos2d中这些元素都只是一个虚构的概念，从属关系并没有这么严格，除了scene必须作为第一层容器之外，sprite
、layer等层级都没有严格的规定，**
比如sprite可以直接插入到一个Scene中，而Sprite也可以插入到另一个Sprite中。

Layer和Sprite也有一定的区别：
> 1. Layer容器默认大小是全屏（即一个全空的Layer默认和屏幕一样大），而Sprite默认无大小，仅当我们用一张图片初始化一个Sprite时，它就会填充得和图片一样大。
> 2. Layer容器的的锚点默认在容器的左下角，而Sprite的锚点则在容器的中点（水平和垂直的中点）

这里讲到一个`锚点`的概念，实际上就是元素布局对齐时的基准点。
说到基准点，首先回到HTML和CSS的世界的定位系统，当我们用 `绝对定位 + left + top` 的组合对div进行定位时，div的锚点（基准点）就是div的左上角啦，而整个世界的定位的坐标系的出发点就是视窗的左上角，这时候我们所定义的left和top值就是div的左上角相对于视窗左上角的距离了。所以按前端习惯的工作流，我们在切设计稿时所测量的距离一般都是元素跟设计稿左上角的距离。

那么回到cocos的世界，不得不说，cocos的定位系统就是很反前端的，首先这个世界的坐标系默认出发点是左下角，也就是说，当我们要对元素进行布局时，默认的参考点就变成设计稿的左下角了。其次，更坑爹的是，当元素是Sprite时，你要量的距离不是设计稿左下角到元素左下角的距离，而是到元素中点的距离。
由于这个锚点会影响到元素的运动和裁切等功能，所以这个设定可能会让习惯了HTML世界开发的前端感到难以接受……

纯天然的Layer默认是透明的，当我们想创建一个带颜色的Layer时，可以继承一个Layer的派生类LayerColor，这样就可以生生成一个带背景色的Layer了。
```javascript
var bg = new cc.LayerColor(cc.color(42, 89, 132));
bg.setContentSize(100,200);
bg.setPosition(0, 0);
this.addChild(bg);
```

另外提一下cc.color这个方法，它可以返回一个cocos中默认的颜色数据，所以你用RGB的值（`cc.color(R, G, B)
`）或者十六进制的形式（`cc.color(#000000)`）都可以生成颜色，非常方便。
此外cocos还提供了类似的一些方法，都会返回cocos默认的数据格式，比如：
		1. cc.p() 返回元素定位数据
		2. cc.size() 返回元素大小数据
		3. cc.rect() 返回要一个矩形选区的数据（类似ps中的选区概念）

## 精灵元素 Sprite

精灵就是游戏中最基本的元素了，常用的精灵包括Sprite（显示图片）、LabelTTF（显示图片）、和MenuItemSprite（显示按钮）。

### 生成Sprite
Sprite生成的方法跟Layer类似，我们在create方法中传入图片的路径，即可得到该图片相对应的Sprite。
注意到create方法中的第二个参数rect，可以有cc.rect(x, y, width,height)获得，rect是一个矩形选区的概念，如果传入该值，那么引擎就会从传入的图片中截取选中区域的图片。
因此，雪碧图在游戏开发时非常好用。
```javascript
this.hand = cc.Sprite.create(oRes['hand']['src'], rect);
this.hand.setPosition(size.width / 2 + 100, oRes['hand']['height'] / 2);
this.addChild(this.hand);
```
### 生成LabelTTF
LabelTTF似乎已经是lite版本中动态显示图片的唯一方案了，在生成一个LabelTTF对象时，我们可以指定它的内容、所用的字体（ttf格式的字体或者系统自带字体）、字体大小、元素所占区域大小、对齐方式等等）;
然后我们可以通过getString方法和setString获取、设置label的内容；
还可以通过setColor设置文字的颜色。
```javascript
this.lbMoney = cc.LabelTTF.create('0', 'Arial', 64, cc.size(200, 64), cc.TEXT_ALIGNMENT_RIGHT);
this.lbMoney.setPosition(size.width / 2  + 50, size.height / 2 + 455);
this.lbMoney.setColor(cc.color('#2a5984'))
this.addChild(this.lbMoney, 1);
this.lbMoney.setString(nData);
```
### 生成按钮
由于引擎基于canvas绘制，所以绘制的元素就不能逐一的绑定通常我们所认知的click、hover等这些html世界中的事件。
所以按钮似乎是cocos世界里唯一可以个简单的一对一绑定点击事件的方法了。
在cocos中按钮又叫菜单，它的结构必须是这样的 菜单（menu） -》 菜单项（menuItem）
因此我们首先要生成1个和多个menuItem，然后再把它（们）添加到menu（类似于Layer）中，这样每个menuItem才能发挥作用。
首先在创建item时，item分为两种方法：
1. MenuItemSprite
```javascript
var sp1 = cc.Sprite.create(oRes1['src'], rect1);
var sp2 = cc.Sprite.create(oRes2['src'], rect2);
var mis = cc.MenuItemSprite.create(sp1, sp2, function(){
	
}, this);
```
2. MenuItemImage
```javascript
var oRes = sResName;
var mii = cc.MenuItemImage.create(oRes['src'], oRes['src'], function(){
	
});
```
所以机智的你一定知道怎样做文字的按钮啦，两个LabelTTF完事：
```javascript
sp1 = cc.LabelTTF.create('start', 'Arial', 64, cc.size(100, 64), cc.TEXT_ALIGNMENT_RIGHT);
sp2 = cc.LabelTTF.create('start', 'Arial', 64, cc.size(100, 64), cc.TEXT_ALIGNMENT_RIGHT);
var mi = cc.MenuItemSprite.create(sp1, sp2, function(){
  console.log(1111);
}, this);
var me = cc.Menu.create(mi);
```

## 动作

cocos中的动作是引擎的一大优势，用起来非常简单，以位移动作为例：
以cc.moveBy方法可以创建一个相对位移的动作，比如这样先相对Y轴唯一40px;
然后FadeIn和FadeOut顾名思义是淡入淡出的动作。
然后注意到spawn方法，这个返回的是多个动作同时执行的效果。
而Sequence方法则可以返回多个动作按顺序依次执行的效果。

当想要在某些动作之后想执行某个方法时，可以用cc.callFunc生成一个callback对象，加入到sequenece对象中即可。

```javascript
var m0 = cc.FadeIn.create(0.3);
var m1 = cc.moveBy(.3, cc.p(0,40));
var m2 = cc.moveBy(.3, cc.p(0, 10));
var m3 = cc.FadeOut.create(0.7);
var m4 = cc.Spawn.create(m0, m1);
var m5 = cc.Spawn.create(m2, m3);
var m6 = cc.moveBy(.3, cc.p(0, -50));
var callback = new cc.CallFunc(function(){
  // todo
}, self);
var m = cc.Sequence.create(m4, m5, m6, callback);
```
## 事件监听和碰撞检测

前面提到过，游戏中事件监听很难单独对某个元素绑定事件，所以得转变一下思路。
当我们要监听某些元素的touch等一系列事件时，
首先我们可以监听到的是整个窗口的touchbegan、onTouchesMoved、onTouchesEnded等事件，
```javascript
cc.eventManager.addListener({
  event: cc.EventListener.TOUCH_ONE_BY_ONE
, swallowTouches: true
, onTouchBegan: function(touch, event) {
    var location = touch.getLocation();
    var oRealLc = self.convertToNodeSpace(location);
    var _x = oRealLc.x;
    var _y = oRealLc.y;
    // sth to do
  }
}, this);
cc.eventManager.addListener({
  prevTouchId: -1
, swallowTouches: true
, event: cc.EventListener.TOUCH_ALL_AT_ONCE
, onTouchesMoved:function (touches, event) {
    var touch = touches[0];
    var location = touch.getLocation();
    var oRealLc = self.convertToNodeSpace(location);
    var _x = oRealLc.x;
    var _y = oRealLc.y;
    // sth to do
  }
, onTouchesEnded: function(touches, event){
    // sth to do
  }
}, this);
```

然后我们可以判断事件的触发点是否在我们要绑定的元素上。
所以问题就变成了，如何判断某个坐标点是否在某个元素显示的区域上，
这个问题其实和检测两个元素是否发生碰撞类似，都会用到一个rectContainsPoint的方法，首先用cc.rect获取要判断的元素的矩形选中区域，然后oPoint为事件触发的坐标点，那么这个方法即可返回oPoint是否再oRect中了（上面的代码中中convertToNodeSpace可以将坐标点转化成指定元素为基准的坐标点）;

```javascript
var oRect = cc.rect(startX, startY, w, h);
var oPoint = cc.p(_x, _y);
cc.rectContainsPoint(oRect, oPoint)
```
## 播放声音
传入参数：声音url、是否循环
```javascript
cc.audioEngine.playEffect(sSoundUrl, false);
cc.audioEngine.playMusic(sSoundUrl, false);
```

## 定时器
定时器也是cocos中一个非常方便的功能，当然也可以用其它方法实现。
```javascript
cc.director.getScheduler().scheduleCallbackForTarget(this, function(){
  // sth to do
}, 1, 0 , 1);
```

## 简单总结优缺点

### 优点
> 1. 框架轻巧，适合web方向的游戏开发
> 2. 系统功能完善，提供了预加载、截取雪碧图、绘图、动画、事件监听管理、定时器等调用方法非常简单的方法

很多缺点都是见仁见智的，这里我只举几个我认为比较硬伤的问题。

### 缺点
> 1. 引擎将需要引入的js文件罗列在一个project.json的文件中，而这个json文件是在引擎启动后再通过XHR加载的，加之引擎本身的版本号管理功能不是很完善，很容易由于现代浏览器强力的缓存机制而造成项目文件不易更替。因此不适合用在需要经常变更版本或代码的游戏上，当然这点可以通过修改引擎源码解决。

> 2. 引擎的预加载方法没有提供回调，当资源加载失败时不能很好的做一些补救措施。举一个实际碰到的问题：在我们的项目中用到了CDN的加速方案，这时候图片其实都是访问CDN获取的，但是由于**某些坑爹的网络提供商的流氓拦截**、**用户使用了某些网络加速器**、**CDN有万分之几的访问失败几率**造成CDN上的图片资源不能正常加载，这个时候引擎就有可能会报错了。而实际上这时候我们更希望引擎能够做一些`回源`的处理，访问源服务器上的资源，这样出错而造成游戏不能玩的几率可能会大大降低。当然这点也可以通过修改引擎解决……

> 3. 自适应方案看上去很厉害，但是在一些特殊运用时还是不尽如人意，比如想在竖屏环境下自动将横版游戏旋转过来，就不是很好做，通过自适应方案得出来的结果也会怪怪的。感兴趣的可以自己试试。

> 4. 声音加载时在安卓手机上有bug，会在加载时把音效播放出来……

### demo地址
*** [DEMO的github地址](https://github.com/shihuacivis/gameCountMoney) ***



**未完待续**

