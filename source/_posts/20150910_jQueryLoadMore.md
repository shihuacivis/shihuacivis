title: jQuery实现滚到底部加载更多
date: 2015-09-08
---

最近在做手机APP上的js，遇到这样的需求，实现难度不大。这个问题可以分为三步解决：
> 1. 判断页面滚到底部
> 2. 然后执行一个ajax请求，获取接下来要展示的内容
> 3. 将内容渲染到事件上。

<!-- more -->

## 监听滚动事件
其中**1**中实现的方法很多，比如：
```javascript
 $(window).scroll(function() {
      if (($(window).scrollTop() + $(window).height() + 250) >= $(document).height()) {
         //执行ajax
}
});
```
## ajax异步引起的重复请求问题
然后就是在**ajax中需要注意一个问题**，因为这个判断**滚动条的方法会执行很多次**，而ajax往往都是异步的，异步执行的方法是不会互斥的，假如滚动时，同时请求3次第二页的数据，那么接下来看到的就是3遍重复的数据了。为此就要引入一个`信号量`的机制，即：
设定变量，如ajaxing = 0，0表示ajax空闲中，1表示正在ajax请求。发送一个ajax前线去判断ajaxing的值，如果为1则return掉。否则就将其置为1，继续执行AJAX，当AJAX返回的数据后再ajaxing = 0。这样就可以保证请求的数据不会重复了。

```javascript
var ajaxing = 0;
function fGetDataAjax(){
  if(1 == ajaxing){
     return;
  }
  ajaxing = 1;
  $.ajax().done(function(msg){
    if(返回数据成功){
      ajaxing = 0;
    }    
	});
}
```
## 拿到数据之后？
第三步很基本功了，拿到数据渲染成html字符串，然后append()到dom中。。。

## 利用缓存
缓存ajax数据是一个提升用户体验的好方法。当用户所要使用的数据不需要十分及时更新时。可以考虑建立一个缓存机制，将每次请求回来的数据缓存起来，下次在请求同一数据时可以优先从缓存里读取，而不再请求数据。

在比较新的浏览器中，我们甚至可以将这些数据存到localstorage中，那么它的作用就更长久了。
比如在我负责的这个模块中，我用键值对将每个关键词与返回数据一一对应存起来。

首先定义一个object用于保存键值对：
```javascript 
var searchCache = {};
```
然后在请求数据时：
```javascript
var key = 关键字;
if(searchCache[key] == undefined){
  //没有缓存，所以要求请求数据了
  $.ajax().done(function(data){
    //返回数据后就可以存进缓存了
    searchCache[key] = data;
    //使用返回的数据
    somethingToDo(data);
  })
}else{
  //有缓存时直接用了
  var data = searchCache[key];
  somethingToDo(data);
}
```
如果你的服务端返回的数据也是json格式的话那么这个方法简直天生一对了，如果这些数据是不常改变的话还可以呀保存到localstorage里长期用了。总的来说要归功于javascript对json数据格式的良好支持吧，json还是很强大的。

`written in 2014/10 本文搬运自LOFTER，让LOFTER纯粹po图！ `