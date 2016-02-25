title: 《JavaScript设计模式》读书笔记-工厂模式
date: 2016-01-11 23:00
---

最近总感觉写的代码没有头绪，很混乱，有时候自己也弄不清楚为什么要这样写。
于是买了一本张容铭写的《JavaScript设计模式》来读读。
希望可以梳理和加深对JS代码的理解。
今天先来理解一下工厂模式。包括`简单工厂模式`、`工厂方法模式`、`抽象工厂模式`
<!-- more -->
## 简单工厂模式

`简单工厂模式（Simple Factory）又叫静态工厂方法，由一个工厂对象决定创建某种产品对象类的实例。`
以我的理解，这种设计模式的目的是：统一几个对象的创建入口，这样就可以抽取这几个对象创建时的共性（共用代码），同时可以由该工厂方法决定具体创建那个对象。

比如屏幕中有modal和pop两种弹出窗体，他们在打开前都需要获取window的width和height属性，那么我们可以通过一个openWin方法实现对两者的调用。

```javascript
  var modal = function(){}
  var pop = function(){}
  var openWin = function(mode) {
    var width = window.width;
    var height = window.height;
    if ('modal' == mode) {
      new model(width, height);
    } else if ('pop' == mode) {
      new pop(width, height);
    }
  }
```

## 工厂方法模式

在上述的模式中有这样一个小缺陷，比方说当系统中新增了一个类dialog，此时我们不仅需要写dialog相关的构造放，还需要在openWin中的判断多加一条，这样静态的方法似乎显得不是很灵活。

而工厂方法模式比之前的简单工厂模式进一步的抽取了共性，不仅是通过一个方法的调用获得两种对象（统一入口），而且要实现两种对象可以通过同一个类（即工厂类）new出来。

```javascript
  var Factory = function(type, content) {
    // 先判断this是否指向Factory
    if (this instanceof Factory) {
      var s = new this[type](content);
      return s;
    } else {
      return new Factory(type, content);
    }
  }
  Factory.prototype = {
    'modal': function(){}
  , 'pop': function(){}
  , 'dialog': function(){}
  }
```

## 抽象工厂模式

`抽象工厂模式`通过对类的工厂抽象使其业务用于产品类簇的创建，而不负责创建某一类产品的实例。
比如在上面的工厂方法中，modal类还分为smallModal和bigModal两种子类，他们都有一个共同的属性type，那我们再把他们的共性抽取出来，作为一个抽象类，不用于实现。

```javascript
  // 抽象工厂方法
  var WinFactory = function(subType, superType) {
    // 判断抽象工厂中是否有该抽象类
    if (typeof WinFactory[superType] === 'function') {
      // 缓存类
      function F(){};
      // 继承父类的属性和方法
      F.prototype = new WinFactory[superType]();
      // 将子类的构造器指向子类
      subType.constructor = subType;
      // 子类原型继承『父类』
      subType.prototype = new F();
    } else {
      // 不存在该抽象类
      throw new Error('未创建该抽象类');
    }
  }
  WinFactory.modal = function() {
    this.type = 'modal';
  }
  WinFactory.modal.prototype = {
    show: function() {
      return new Error('抽象方法不能调用');
    }
  }

  // 生成一个smallModal子类
  var smallModal = function() {
  } 
  WinFactory(smallModal, 'modal');
  smallModal.prototype.show = function() {
    return 'show';
  }
  var modal = new smallModal();
  console.log(modal.show());
  console.log(modal.type);
```