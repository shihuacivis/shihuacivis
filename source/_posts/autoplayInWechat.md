title: 在ios的浏览器中audio标签不能autoplay的解决方案
---

As we knew, ios为了安全性屏蔽了audio标签的autoplay属性，需要由其它事件触发调用才能播放声音。
那么，浏览器中能触发的事件就很多了。
最早我们的实现方式是当用户第一次touch屏幕时去吃饭audio.play() 方法。后来发现这个方法有缺陷：假如用户不去点屏幕，那么声音就不会播放了。
一个比较灵活的方法，就是去监听页面上的某张图片的onload事件，然后去触发play()方法，就可以实现autoplay的效果了。
因为在H5的页面中，我们往往需要预加载多张图片之后才显示页面，正好可以顺带触发play方法，一举两得。
```javascript
img.onload = function() {
  audio.play();
}
```