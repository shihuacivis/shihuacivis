title: iphone4下audio标签占据空间的BUG
---

前段时间在"指导"某学弟做H5专题页面时遇到iphone4的微信访问页面时内容莫名塌陷的问题，而且在chrome中模拟不出来。后来发现是页面中的audio标签竟然占据了一定的高度，于是造成了内容塌陷。解决的方法很简单。用样式将其隐藏起来即可。
```css
audio {
  width: 0;
  height: 0;
  display: none;
}
```