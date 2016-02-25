title: 初识arrayBuffer和typeArray
date: 2015-12-21
---

之前很少接触到这么冷门的知识点，直到最近有一个项目，需要前端通过websocket与服务端连接，服务端直接向前端发二进制的数据包，我才知道js还有这么arrayBuffer和typeArray这两货。我们可以用他们来解析服务端发来的二进制包。

<!-- more -->

## 知识回顾

本来以为从事前端就再也不会接触进制、字节等这些晦涩的概念。

没想到苍天饶过谁……

### 二进制包的相关概念

一般服务端会事先跟我们约定好发来的二进制包的格式，我们称之为`协议`

首先，一般一条二进制包数据会包含多个`字节`，光凭字节串我们是不能解析出里面的数据含义的。

**协议**的作用在于约定了二进制包的数据格式。我们通过协议可以知道二进制串中，从第几个字节到第几个字节表示什么类型数据，从而得出其准确的值。

回归正题，我们先了解一下如何从websocket中读取二进制数据。

## 如何读取二进制数据

**二进制数据是不能直接像其它js的基本数据类型一样直接通过等号获得和显示的。**

我们需要用到`FileReader`对象的`readAsArrayBuffer`方法读出其中的所有字节并将其转换成js能读取的arrayBuffer数据对象。

```javascript
ws.onmessage = function (e) {
  var message = e.data;
  blob = message;
  var fileReader     = new FileReader();
  fileReader.onload  = function() {
      // this.result为读取到的二进制包内容
      arrayBufferNew = this.result;
  };
  fileReader.readAsArrayBuffer(blob);
}
```

ArrayBuffer会在内存中新建一块缓冲区域，用于装载原始的数据包，
我们可以理解成，它创建了一个缓冲数组，然后把原始数据包的每个`字节`的值单独存放于一个数组元素中，但这个数组并不能直接进行读取。
我们还需要新建一个`TypeArray`对象，来读取这个缓冲区域的数据。

`TypeArray`包含下面几种类型。

| 类型          | 大小  |  描述 | C语言中的等效类型 |
| -------------|:----:| -----:| ----------------:|
| Int8Array    |  1  | 8-bit | signed char |
| Uint8Array   |  1  | 8-bit | unsigned char  |
| Int16Array   |  2  | 16-bit | short |
| Uint16Array  |  2  | 16-bit | unsigned short |
| Int32Array   |  4  | 32-bit | int |
| Uint32Array  |  4  | 32-bit | unsigned int |
| Float32Array |  4  | 32-bit | float |
| Float64Array |  8  | 64-bit | double |

**重点来了**

以一个例子为例,首先协议体如下：

```c
unsigned char flag;
unsigned short id;
unsigned int num;

```
当我们收到如下面这个字节包时:

```javascript
0x01  0x02   0x03   0x04   0xFF   0xDD   0x4c 0x8f
```
我们知道，一个字节占8位，那么我们可以用Uint8Array读取占1字节的数据，可以用Uint16Array读取占2字节的数据。

那么我们可以用Uint8Array来显示数据的每个字节的内容：
```javascript
arrayBufferNew = this.result;
arr  = new Uint8Array(arrayBufferNew);
console.log(arr);
```

根据协议体，我们知道前2个字节即为flag（char型占2字节），第3-4个字节是id（short型占2字节），而后4个字节则为num（int型占4字节）。


那么读取数据的过程如下： 

```javascript
fileReader.onload  = function(progressEvent) {
  arrayBufferNew = this.result;
   // 从第1个字节索引开始读取1个Uint16Array（即16位，2字节）
  cFlag  = new Uint16Array(arrayBufferNew, 0, 1);
  // 从第3个字节索引开始读取1个Uint16Array（即16位，2字节）
  sID = new Uint16Array(arrayBufferNew, 2, 1);
  // 从第5个字节索引开始读取1个Uint32Array（即32位，4字节）
  nNum  = new Uint32Array(arrayBufferNew, 4, 1);
  console.log(cFlag[0],sID[0],nNum[0]);
}
```

以上我们就完成了一条协议数据的读取。

## 字符串和arrayBuffer的转换

如何快速对字符串和arrayBuffer进行转换：

```javascript
function ab2str(buf) {
  return String.fromCharCode.apply(null, new Uint16Array(buf));
}
function str2ab(str) {
  var buf = new ArrayBuffer(str.length * 2); // 2 bytes for each char
  var bufView = new Uint16Array(buf);
  for (var i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

然后，我们就可以将生成的arrayBuffer转成Blob对象（即二进制流），发送给后台：
```javascript
var ab = str2ab('test');
var blob = new Blob([ab]);
```

当然，一般我们在做websocket通讯时还会对数据进行加密，我自己目前也正深陷加密的无底洞中……等从洞里爬出来再总结一下加密相关的内容……