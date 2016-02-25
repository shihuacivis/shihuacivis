title: 微信H5中video常见的问题
date: 2016-02-23 17:00
---


最近帮朋友看了微信H5播放video，实现类似吴亦凡参军H5的效果。在X5引擎中遇到了全屏播放、自动播放等几个问题。
<!-- more -->

## 1.视频全屏播放

我的思路是用一个宽高均为100%的容器`.video-wrap`来包住`video`，再让`video`的高度等于容器的高度，进而实现全屏。

不过这里遇到了2个问题：
- 在ios上，微信会默认调用原生的播放器来全屏播放视频，会出现白色的菜单和进度条，影响全屏的体验。
- 在安卓上，微信会调用X5引擎底层播放器组件来播放视频，容器的z-index属性对其无效。此时也会出现菜单，而且最后还会无耻的出现“推荐视频”一栏…… 据说如果播放的是qq.com域名下的视频则不会碰到这个问题。[X5内核视频播放官方问题集](http://x5.tencent.com/guide?id=2009)

在ios中，我们用`webkit-playsinline`可以让webview使用H5的video直接播放，而不再出现菜单和进度条。

```html
  <div class="video-wrap">
    <video id="video" height="100%" x-webkit-airplay="true" webkit-playsinline="true" preload="auto">
      <source src="res/video.mp4">
    </video>
  </div>
```

```css
.video-wrap {
  position: absolute;
  z-index: 2;
  width: 100%;
  height: 100%;
  background: url(../images/p2/bg.jpg);
  background-size: cover;
  /*display: none;*/
}
video { font-size: 100%; line-height: 0; vertical-align: baseline; -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; outline: none; }
```

## 2.自动（定时）播放视频

在ios下，我们可以直接调用`video`的play方法实现视频的播放

```javascript
  setTimeout(function(){
    // 原生
    document.querySelector('#video').play();
    // jQuery
    $('video')[0].play();
  }, 1000);
```

但在安卓下，由于X5内核绑架了`video`标签，而且要求视频首次播放时，必须是由用户亲自点击事件触发的才能播放，所以setTimeout不能调用play方法，并且也不能用js去触发click事件……[X5内核视频播放官方问题集](http://x5.tencent.com/guide?id=2009)
解决的办法：
A.改成用户点击某个按钮后开始播放。
B.在游戏开始按钮时，先播放视频然后马上暂停，等到正式要播放视频时，就可以实现跟ios一样的调用方法了。


## 3.监听视频播放完毕

当视频播放完毕后我们希望它能自动隐藏掉并跳到其他界面。
video提供了一个视频播放完毕后的事件`ended`

```javascript
  document.querySelector('#video').addEventListener('ended', function(evt) {
    document.querySelector('.video-wrap').innerHTML('');
  });
```
