title: prototype.function与this.function的区别
date: 2016-01-08 1:00
---

当我们在用javascript中的function做面向对象开发时，可能会有这样的疑惑，
prototype.function与this.function都可以定义对象的方法，那二者有何区别呢？
<!-- more -->
## 初步分析

先用一段代码来说明二者的区别：

```javascript
  var actor = function() {
    var param = 1;
    this.act = function() {
      return param;
    }
  };
  actor.prototype.play = function() {
    return typeof param;
  };
  var man1 = new actor();
  var man2 = new actor();
  console.log(man1.act === man2.act);   // false
  console.log(man1.play === man2.play); // true

  console.log(man1.act()); // 1
  console.log(man1.play()); // undefined

```

## 方法指向的内存地址不同
此时我们会发现，man1、man2虽然是同一个『类』new出来的对象，但是他们的`act`方法却不相等，也就是说，两个对象的的act方法的各自占据一块内存区域。
而用`prototype`定义的`play`方法，则指向单一的同一块内存区域。

**由此可见，如果一个对象被实例化多次，prototype定义的方法相对会更节省内存。**

## 闭包内的参数
当闭包中的param参数没有提供外露句柄时(如this.param = XXX)，play方法是获取不到它的。

## 如何取舍

1. 如果我们定义的方法里经常需要调用闭包中的变量（如上面的param），这时候使用this.function会更加便捷。
2. 当我们定义的类会生成多个对象，而对象中的某些方法，执行的过程中不需要太多的访问闭包中的参数，从性能优化的角度会优先考虑prototype。
3. 当在单例模式下（即一个对象在程序中只保持一个实例），这时候两者定义的方法从内存消耗上基本上是持平的，因此主要参考第一条中的原则。


