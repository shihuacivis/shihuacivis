title: 此处功能日后必改，不要写死
date: 2015-11-18
---

最近有一张神图，大意是产品说『此处功能日后必改，不要写死』

间接的吐槽了产品狗就是喜欢糊逼改需求，因此我们的代码千万不能写成hardcode。

也许你曾经听过hardcode，有的人用它来形容很难懂的代码，很形象，你的代码很硬，所以不能适应变化。

而这里我想说的是一种编码习惯：

**将程序体中要调用的变量直接写死代码中。**


<!-- more -->

比如我们要计算一个边长是100的正方形面积：
```javascript
var a = 100 * 100;
```
因为我们会习惯性的认为100是一个确定的值，我们就直接用数字里表明，而不是一个变量。

那么这个时候就会带来两个问题：

1. 100这个数值的含义不清楚。当代码交到别人手上，或者过了一段时间后自己再回来看时，可能就不清楚这个100是什么意思了。这就变成了传说中的`magic number`；
2. 如果有一天，正方形的边长变了，是不是就要把数值改变了呢？假设这个边长在程序中多个地方进行运算，与此同时如果程序中有其它变量的值也是100，那么改起来就头疼了；


所以一个良好的编程习惯就是少用hardcode。这会让你的代码质量（可读性、复用性等）有一个质的飞跃

**注意，避免hardcode是一种编程的思想，而不是简单的把数值变量化**


## 目的

首先要先清楚去除hardcode的目的：

> 1. 让代码更加语义化。让自己/他人在阅读（审查）代码时都能很快的看懂某一个变量、方法等的意思；
> 2. 提高代码的复用性。在去除hardcode的过程中，其实我们已经将数值相同的变量的入口提取出来（抽取共性），这样就只需要一处修改、多处受用了。
> 3. 提高效率。避免在编码过程中认为输错某些字符而出错……



## 几个经典的应用：


### ajax的url

很多时候某个请求地址的用途很难从url字面看懂（如a.php），如果我们将其提取出来，放到一个oUrl['getUserInfo']对象中。
那么首先它的作用就可以从字面上理解了，getUserInfo嘛。
其实还有一个更大好处，在实际项目中，当前后端分工明确时，前端常常需要自己测试ajax模块，而不是直接去调实际PHP给出的接口（又或者说前端往往先于PHP开发），当我们在本地测试时，ajax的请求地址就和线上的不同了。
如果我们采取这种写法，将地址的配置参数暴露出来，当我们提交给PHP后，他只需要修改这几个配置项就可以了。而不需要再改我们的js代码了。
假如程序中很多个（假设10个）模块都需要访问a.php调数据，那么如果有一天，接口的地址换了（比如上面这种前后端分离后前端交付js给php后的情景），那么我就不需要10个模块都一一的去改。

```javascript
var oUrl = {
  'getUserInfo': 'a.php'
, 'setUserInfo': 'b.php'
};
$ajax({
  'url': oUrl['getUserInfo']
, 'type': 'GET'
});
```

### 事件绑定

ajax的例子中已经把优点说的很明显了，下面再分享一种在事件绑定中的应用：
我们知道，在pc端我们的点击事件一般都是绑定click，而移动端我们为了更快的响应则往往绑定touchstart事件，于是就会发现在pc端上调试touchstart事件不太方便而在移动端click又不是我们想要的结果。

因此，我们把要绑定的事件抽取成变量，先判断系统环境然后再给变量赋值，这样就可以快捷的在pc和手机上切换啦。(判断方法仅供参考)

```javascript
function fCheckMoblie()  
{  
  var aUserAgentInfo = navigator.userAgent;  
  var aMoblieAgent = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod'];
  var bFlag = false;  
  for (var i = 0; i < aMoblieAgent.length; i++) {  
     if (aUserAgentInfo.indexOf(Agents[i]) > 0) { 
      bFlag = true; 
      break; 
     }  
  }  
  return bFlag;  
} 
var bMobile = fCheckMoblie();
var sClickType = bMobile ? 'touchstart' : 'click';
$('#aaa').on(sClickType, function(){
  // todo
})
```

### 思路拓展 —— 抽取方法（代码片段）

上面两个案例可以看到将变量提取出来的优点，其实这样的思路不仅对于变量受用，对于方法（或者是一整段的逻辑代码）也同样适用。

我们刚开始接触一些业务复杂的js时可能都会写过下面几种代码：

* 1. 超级多的if else
```javascript
functon aaa(sCity) {
  if ('shanghai' == sCity) {
    // 这里是50行逻辑代码代码
  } else if ('beijing' == sCity) {
    / 这里是50行逻辑代码代码
  } else if ('guangzhou' == sCity) {
    / 这里是50行逻辑代码代码
  } else if ('hangzhou' == sCity) {
    / 这里是50行逻辑代码代码
  } else if ('nanning' == sCity) {
    / 这里是50行逻辑代码代码
  }
}
```
上面这种一个分多种情况处理的逻辑代码，假设每个情况50行，当你写完后要进行调试修改代码时发现滚轮内心几乎是崩溃的。这种现象很常见，因为和有可能在开始编码时不能预见逻辑会如此复杂。但客观的说这样的代码可读性略差。

**有的人会说这种情况换 `switch case` 会更好**
但其实如果每种情况要里有几十行上百行代码时这么写其实是换汤不换药的。

一种更为推荐的写法，假如只有`shanghai` 、 `beijing`等这5种情况，那么我们先将这这些情况的逻辑代码抽取出来：

```javascript
oFunc = {
  'shanghai': function(){
    // 上海的50行
  }
, 'beijing': function(){
    // 50行
  }
, 'guangzhou': function(){
    // 50行
  }
, 'hangzhou': function(){
    // 50行
  }
, 'nanning': function(){
    // 50行
  }
}
function aaa(sCity) {
  oFunc[sCity]();
}
```

这时候aaa这个方法就轻便了很多了，而原来数十行的代码段也有了自己的方法，这时候就可以多处去调用相同的代码片段了。如果这时候你下意识的想去看看oFunc中的五个方法是不是再可以抽取共同的模块（代码片段）那么基本上就已经溜的飞起了。

* 2. 重复写很多相似的代码
当页面中要处理很多元素时，就经常会遇到这种情况。

```javascript
var a = new Person();
a.name = 'name1';
a.gender = 'man';
a.age = '18';

var b = new Person();
b.name = 'name1';
b.gender = 'man';
b.age = '18';

var c = new Person();
c.name = 'name1';
c.gender = 'man';
c.age = '18';
```

其实处理的这三个元素所进行的工序是一毛一样的。一般遇到这种情况，超过三段相同的代码时就可以考虑进行封装了。

```javascript
funtion fCreatePerson(name, gender, age) {
  var o = new Person();
  o.name = name;
  o.gender = gender;
  o.age = age;
  return o;
}

var a = fCreatePerson('name1', 'man', '18');
var b = fCreatePerson('name1', 'man', '18');
var c = fCreatePerson('name1', 'man', '18');

```

上面这种情况普遍出现在某些匿名函数中，比如：

```javascript
$('#btn1').on('click', function(){
  a = $(this).data('abc');
  alert(a);
});

$('#btn2').on('click', function(){
  a = $(this).data('abc');
  alert(a);
});

```
刚开始往往会被匿名函数牵着鼻子走，习惯性的把逻辑代码都写在这个匿名function中。
而实际上，按照本文的思路，这段匿名函数其实可以抽出来写……

### 脑洞大开 CSS也行？
其实这个是胡扯啦，css这种静态的stylesheet基本上不存在变量这种说法啦~
基本上就是你在css里定义了多少，那么页面上就显示多少了。
不过直至rem的出现。
rem在这里就可以看成是一个预设的变量啦，它的默认值是根元素的font-size。
当页面加载后，你可以修个font-size的大小，这时候，所有基于rem设置大小的值都会响应啦。
是不是很叼？？？

