title: 移动端判断浏览器是横竖屏方向
---

有时候，我们做一些H5特效页或游戏时，只做了竖屏状态下的设计和实现，此时我们并不希望用户以横屏状态打开，因此我们要在页面加载之初判断手机的横竖屏方向，做相应的处理。
在现代浏览器中，我们可以通过window.oritentation判断屏幕的横竖屏方向，当：window.oritentation = 0 或者 180 时，则手机为竖屏状态，反之则为横屏。

然而，在一些不支持window.oritentation属性的环境下，我们可以判断window的width和height来判断手机的横竖屏状态（通常状态下，竖屏时屏幕高度大于屏幕宽度）

在下面的代码中，还定义了一个_initScreenDir = 0的初始方向，当屏幕从横屏转为竖屏时，通过监听orientationchange的方法，如果不支持该事件的情况下，则监听resize事件，会自动刷新页面以保证页面的初始化操作正常（如页面布局，rem的设置等等）。

```javascript
_initScreenDir = 0;
function _onorientationchange(e){
  $el = $("#tips");
  if (window.orientation == 0 || window.orientation == 180) {
    $el.hide();
    _initScreenDir == 1 && (window.location.reload());
    return false;
  } else if (window.orientation == 90 || window.orientation == -90) {
    $el.show();
    _initScreenDir = 1;
    return false;
  }
  var width = $(window).width();
  var height = $(window).height();
  if(width < height){
    $el.hide();
    _initScreenDir == 1 && (window.location.reload());
    return false;
  }else{
    $el.show();
    _initScreenDir = 1;
    return false;
  }
};
_onorientationchange();
window.addEventListener("onorientationchange" in window ? "orientationchange" : "resize", function(e){_onorientationchange(e);}, false);
```