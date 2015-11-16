title: js中的数据存储
---

在日常的开发中经常会遇到需要将临时数据混存起来供后期使用的场景，特别是做一些管理系统或者游戏时，都会要处理大量的数据。这时候就需要思考如何使用js来存取数据了。
## 利用简单的变量

去年五月份刚开始实习的时候，对于js的数据类型理解还只是一知半解。所以基本上是这样解决的：
假设页面上我要存取一个用户的`名字`、`性别`、`生日`,那么我就
```javascript
var sName = 'shihua';
var nGender = 1; // 是的！我是「gender」党并长期对「sex」党以及「登陆」党表示关心
var sBirth = '1/19';
```
于是，当页面上需要存两个人的数据时，就`临时性思维定势`的依样画葫芦，再多定义一个变量：
```javascript
var sName1 = 'ayu';
var nGender1 = 0;
var sBirth1 = '2/19';
```

**那么问题就来了**
你会发现这样的变量存取都是1对1的，也就是1个js变量只能存1个用户数据项。
如果有30个用户名怎么办，是不是要预先设定30个sName变量呢？
又比如，用户数据是ajax动态拉取的，有可能1条也有可能100条，那应该预设多少变量呢？
所以这么暴力的方法显然不可取……
于是就开始思考有什么方法可以让一个变量存取多条数据，于是我想到数组。

## 数组的运用

利用数组可以非常简单的管理批量数据的增删改查。于是代码变成了这样：
```javascript
var aName = [];
```
### 1. 插入基本数据类型
当要插入一个新用户的名字（基本数据类型）时，就把它的名字push到aName中：
```javascript
var sName = 'test';
aName.push(sName);
```
*那么问题又来了*
这样看似解决了上述动态变量数的问题，但需求中有`名字`、`性别`、`生日`三个类型的变量数据，那是不是要创建三个数组来分别存取呢？
试想一下，如果要存取的数据类型有100种而非三种呢，是不是又碰到了和第一步中类似的困境。
另外，倘若只有三个数据类型，分别用三个数组存取的话，当你要增、删、改数据时是不是要同时操作三个数据？是不是心很累还容易出错？
这个相对暴力思路显然又不可取了。

### 2. 插入引用数据类型
然后度娘告诉我们，数组不仅可以存取基本数据类型，还可以存取引用类型。
另外，引用类型中，一个obeject可以包含多个字段，于是插入新用户数据时就变成这样：
```javascript
var aUser = []; // 用这个数组来存
var sName = 'shihua';
var nGender = 1;
var sBirth = '1/19';
var oUser = {
      'name': sName
    , 'gender': nGender
    , 'birth': sBirth
    , 'age': 23
    }
aUser.push(oUser);
```

于是乎，每个用户的所有信息单独对应一个对象，而所有对象又只存在一个数组对象中……
然后就可以轻松做下列操作了：
```javascript
aTen = aUser.slice(0, 9) // 获取前十个用户的信息
aNew = aUserGA.concat(aUserGB) // userGA 和 userGB的数组合并为一组
```
数组的api有很多，在此不一一列举。
这样看起来似乎以及可以愉快的玩耍了。

**直到有一天产品经理说她想根据用户的年龄进行排序**
### 3. 数组的排序
这个问题其实就是如何根据数组中的对象中的某个属性进行排序。其实js的sort方法很好用，对基本数据类型的排序就不必说了，根据引用类型中的属性值排序的方法如下：
```javascript
// 递减排序
var fSortByAgeDESC = function(oA, oB) {
  return +oB['age'] - +oA['age'];
}
// 递减排序
var fSortByAgeASC = function(oA, oB) {
  return +oA['age'] - +oB['age'];
}
aUser.sort(fSortByAgeDESC);
```

**直到有一天产品经理说她想通过用户名查询用户的详细数据**
### 4. 数组的查询和不足
然后你就发现数组似乎只能够通过索引index来访问数据，如aUser[1]来访问数组中第二个对象的数据。
如果要实现产品经理的要求，可能就需要for循环一下，寻找符合条件的数组：
```javascript
var sName = '要找的用户名';
for (var i = 0; i < aUser.length; i++) {
  var o = aUser[i];
  if (sName == o['name']) {
    return o;
  }
}
```

## 利用object存取数据
利用上面循环遍历看起来似乎有些麻烦，然后就看到object可以通过object['name']这样方便的取到数据，于是就考虑是不是可以用object来存取数据。
```javascript
var oUserData = {
  'shihua': {
      'name': 'shihua'
    , 'gender': 1
    , 'birth': '1/19'
    , 'age': 23
  }
}
var sName = '要找的用户名';
var oData = oUserData[sName]; // oData 就是要找的数据
if ('undefined' == typeof(oData)) {
  // 若干数据未定义就说明该用户不存在
}
```
这样看起来是不简单多了，另外当需要循环遍历出所有用户的信息时，可以用for in循环:
```javascript
for (var key in oUserData) {
  var sName = key; // key就是oUserData中的「键」
  var oData = oUserData[key]; // oData就是每个用户的信息
}
```
*看起来object比array更适合查询数据。*
> array还是有优势的： 
1. array可以通过array.length快速获取数组元素的个数，而object中则没有类似的属性值，需要通过别的方法获取，略显麻烦
2. 当要从数据集合中删除某条元素是，array可以很干净抹掉，而object中的「键」则不易清理
3. object中的数据排序方法相对麻烦
4. 数组数据更易于复制、转移


------
>  object的优势则主要在于：
1. 查询方便，根据键即可定位到数据对象
2. 在一个object中，一个键值只能对于一个数据，因此可以避免数据重复（反过来说如果是用数组存数据时，如果需要确保数据不重复，那么在push数据之前就需要先判断一下数组中是否已经存在）
*因此在项目中，要根据数据的性质来决定使用数组或者object，也可以将两者结合起来，实现更复杂的应用场景*


## 关于复制引用类型数据

经常会遇到需要复制一个array或者object的需求。
如果是简单粗暴的 a = b 这样复制那就可能出问题了，你会发现当b中数据发生变化时， a的数据竟然也改变了，这也许并不是你想见到的结果，因为 a = b 这样对引用类型赋值时，a实际上等于b对象的一个内存地址，也就是说，访问a时实际上访问的是b。
正确复制引用类型的方法如下：
1. 复制array
```javascript
// 方法一 简单粗暴循环生成新数组
var arr = [];
for (var i = 0; i < aUser.length; i++) {
  arr.push(aUser[i]);
}
// 方法二 用concat
var arr = aUser.concat()
```
2. 复制object
当object中存的数据为基本数据类型时，可以使用浅拷贝。反之，如果object中有很多层数据时，若使用浅拷贝会发现其子层还是会遇到之前发生的问题
```javascript
// 循环复制
var obj = [];
for (var key in oUserData) {
  obj[key] = oUserData[key];
}

// 深拷贝就是要拓展一个递归的方法，去一层一层遍历，当子层数据类型为object时递归遍历下一次，直至最后一层。网上很多代码的。不贴了……

```