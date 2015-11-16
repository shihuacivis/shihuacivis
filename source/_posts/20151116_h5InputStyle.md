title: 实现移动端自定义背景的input输入框
---

input的特殊之处在于它有预设的行高，背景，边框等属性。
当我们要做一个完全自定义背景的输入框时，我们首先要把它所有的默认背景、边框、背景都去掉，然后将背景设置成我们想要的效果。

在实际的移动项目中，我们会发现这样设置的input被focus时，光标非常的大（高），几乎沾满了一行。这显然不是我们想要的效果，还需要加上appearance:none这个属性，它的作用是将input默认的属性样式去掉。

另外input中不需要再设置line-height了，否则会画蛇添足。

```css
.input-wrap input {
  display: block;
  width: 5.06rem;
  height: .96rem;
  margin: 0 auto;
  border: none;
  background: none;
  color: #602f00;
  vertical-align: middle;
  font-size: .48rem;
  -webkit-appearance:none; /* Safari 和 Chrome */
  appearance:none;
}
```