title: 判断页面是否在移动端中打开
---

在现代浏览器中，我们可以通过判断window的navigator对象的userAgent取到客户端的信息，从而判断是否是移动端，判断脚本如下：

```javascript
if(/iphone|ios|android|mobile/i.test(navigator.userAgent.toLowerCase())) {
    // 在移动端打开
}
```

同理，判断页面是否是在微信中打开

```javascript
if(/micromessenger/i.test(navigator.userAgent.toLowerCase())) {
    // 微信内置浏览器
}
```