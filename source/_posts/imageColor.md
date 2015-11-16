title: 在高级浏览器中获取图片的边缘颜色
---

有一个我很喜欢的网站——[网易公开课](http://open.163.com)，记得五月份帮小雪姐做微课视频网站的时候参考过它。有一个效果我很喜欢，就是`首页的banner轮播组件的背景会根据图片的颜色改变，让其跟图片的边缘同颜色`，这样图片看上去就像自然的延伸到屏幕边缘。很不错。

当时有想过怎么实现，觉得可能是要后台读取图片，拿到它边缘的颜色，然后再告诉前端。这个放在PHP是可以办得到的（吧）。

后来有看到一个纯前端的解决方案，不过要在高级浏览器中才能用（网易公开课也是全面嫌弃低版本浏览器的）。这里用到canvas中的一个方法。

解决的思路是做一块隐藏的canvas，在它上面draw上要获取的图，然后用getImageData方法去获得图片边缘的某一点上的RGB信息。这样就能拿到图片边缘的颜色啦。

核心代码：
```javascript
var canvas = document.getElementById('myCanvas');
if (canvas.getContext) {
	var ctx = canvas.getContext('2d');
}
ctx.drawImage(image, 0, 0, 800, 600);
var imageData = ctx.getImageData(0, 1, 1, 1);
```
imageData是一个数组，imageData[0]、imageData[1]、imageData[2]分别就是其的R、G、B的值啦

`written in 2014/10 本文搬运自LOFTER，让LOFTER纯粹po图！ `