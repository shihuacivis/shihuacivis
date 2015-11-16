title: js中的function小记
---

最近参加了一个js方面的培训，讲师用两天时间给我们回顾和剖析了function的种种，的确有所收获，小记于此。


## function是js中的一等公民

我们会发现js中一些基础的对象实质上也是function

```javascript
console.log(typeof Number)
console.log(typeof String)
console.log(typeof Function)
console.log(typeof Object)
```

## function用于面向对象

说到对象，我们首先想到要有一个类，然后把类进行实例化得到对象。
而function就是我们所要的类（class）了。
比如下面这段代码，我们定义了一个Person类，然后传入`字面量`参数实例化了一个对象——sh。

```javascript
function Person(opt) {
	this.name = opt.name;
	this.gender = opt.gender;
	this.skill = function(){};
}
var sh = new Person({name:'shihua', gender: 'man'});
console.log(sh);
console.log(typeof sh);
```
通过console可以看到新生成的sh其实是一个Object,然而细心的人会发现，它除了我么自己定义的属性方法外，还多了一个`__proto__`的对象。这个就涉及了function中的一大精华原型链了。

## js中的原型链
关于js中的原型链网上已经很多详细的解释了，之前也大概知道可以用prototype的方法给对象添加方法，比如：

```javascript
function Person(opt) {
	this.name = opt.name;
	this.gender = opt.gender;
}
Person.prototype.skill = function() {};
```

*这样Person就具有skill方法。但其实用下面这种方式定义，也可以达到目的。*

```javascript
function Person(opt) {
	this.name = opt.name;
	this.gender = opt.gender;
	this.skill = function(){};
}
```

所以我一直不是特别明白prototype方法的意义。
但其实细心的人会发现，用这种方法时，skill所指向的方法的内存地址是不一样的。
也就是说，当我们用后一种类时，每创建一个对象，都会在内存中新创建一个skill方法。这样显然是很影响性能的。
然而，如果是用prototype添加的skill方法，指向的内存地址则是一致的。

```javascript
function Person(opt) {
	this.name = opt.name;
	this.gender = opt.gender;
	this.skill = function(){};
}
var a = new Person({name:'aa', gender: 'a'});
var b = new Person({name:'aa', gender: 'a'});
console.log(a.skill === b.skill);

function Human(opt) {
	this.name = opt.name;
	this.gender = opt.gender;
}
Human.prototype.skill = function() {};
var a1 = new Human({name:'aa', gender: 'a'});
var b1 = new Human({name:'aa', gender: 'a'});
console.log(a1.skill === b1.skill);
```

显然prototype才是更佳的做法。
那么就得说到prototype的运作原理了，当我们new一个对象时，会赋予其一个`__proto__`的属性，用于指向它的基类，使其具有基类的方法的指针，而子类自身实质上是不具有这些方法的。
当我们访问子类时，若其本身不具有这个属性和方法，则会根据`__proto__`向上一级去寻找，若上一级也不具有，则会依此去更上一级搜寻。
而prototype就是自主配置`__proto__`的入口。
这样就不会造成内存浪费，而对象方法也更易于被`继承`。


## js中实现extend

有时候我们想在某个类的基础上派生出一个新的类，使其具有新的属性方法，同时不影响原有类。在java中我们把这个方法叫extend。在js中，extend的实现方法如下，klass就是我们要的具有extend功能的类

```javascript
var klass = function(opts) {
};
klass.extend = function(opt) {
	var child = function(){};
	var F = function() {
		this.constructor = child();
	};
	F.prototype = this.prototype;
	var o = new F();
	child.prototype = o;
	// 拓展新方法
	for (var attr in opt) {
		child.prototype[attr] = opt[attr];
	}
	child.prototype.__super__ = this.prototype;
	return child;
}
```
