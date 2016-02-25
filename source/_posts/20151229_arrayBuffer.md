title: TypeArray、ArrayBuffer、Blob的相互转换
date: 2015-12-29 20:00
---

- `Blob`是现代浏览器中提供的能够装载二进制流（文件）的容器对象。
- `ArrayBuffer`是能够装载`Blob`（二进制流）数据的原始缓冲区，`ArrayBuffer`不能直接通过js读写。
- `TypeArray`是`ArrayBuffer`的一种类数组的视图对象，可以将`ArrayBuffer`按不同字节数读取成类似数组形式的数据类型，从而可以向读写数组元素一样，实现对`ArrayBuffer`数据的读写。常见的`TypeArray`包括`Uint8Array`,`Uint16Array`,`Uint32Array`等。[点这里查看所有的TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)

所以我对三者的理解是： `Blob` <-> `ArrayBuffer` <-> `TypeArray` <----> `Array`
由于`TypeArray`和`Array`有些相似，因此往往我会选择在`TypeArray`这层做处理。
下面是`TypeArray`、`ArrayBuffer`和`Blob`之间相互转换的方法。

<!-- more -->

## Blob to ArrayBuffer （二进制流转ArrayBuffer）

```javascript
/* websocket的情况下二进制流的获取 */
var svip = 'websocket';
var ws = new WebSocket(svip);
ws.onmessage = function (e) {
var message = e.data;
}


var blob = message;
var fileReader = new FileReader();
fileReader.onload  = function(progressEvent) {
  arrayBuffer = this.result; // arrayBuffer即为blob对应的arrayBuffer
};
fileReader.readAsArrayBuffer(message);
```

## ArrayBuffer to Blob （ArrayBuffer转Blob）

```javascript
var ab = new ArrayBuffer(32);
var blob = new Blob([ab]); // 注意必须包裹[]
```

## ArrayBuffer to Uint8 （ArrayBuffer转Uint8数组）

Uint8数组可以直观的看到ArrayBuffer中每个字节（1字节 == 8位）的值。一般我们要将ArrayBuffer转成Uint类型数组后才能对其中的字节进行存取操作。

```javascript
var ab = arrayBuffer; // arrayBuffer为要转换的值
var u8 = new Uint8Array(ab);
```

## Uint8 to ArrayBuffer（Uint数组转ArrayBuffer）

我们Uint8数组可以直观的看到ArrayBuffer中每个字节（1字节 == 8位）的值。一般我们要将ArrayBuffer转成Uint类型数组后才能对其中的字节进行存取操作。

```javascript
var u8 = new Uint8Array();
var ab = u8.buffer; // ab即是u8对应的arrayBuffer
```

## Array to ArrayBuffer（普通数组转ArrayBuffer）
```javascript
var arr = [0x15,0xFF,0x01,0x00,0x34,0xAB,0x11];
var u8 = new Uint8Array(arr);
var ab = u8.buffer;
console.log(ab); // ab为要解析的ArrayBuffer
```

## 获取/设置ArrayBuffer对应的数值

一串ArrayBuffer是可以被“理解”为很多个值的，以下面这个值为例，

按照服务端的协议，这串数据流的格式如下：
1 unsign byte (1字节) + 1 unsign int (4字节) + 1 unsign short (2字节)

```javascript
var arr = [0x01,0x02,0x00,0x00,0x00,0x00,0x03];
var u8 = new Uint8Array(arr);
var ab = u8.buffer;
console.log(ab); // ab为要解析的ArrayBuffer

var u8 = new Uint8Array(ab, 0, 1); // (arraybuffer, 字节解析的起点, 解析的长度)
var val_byte = u8[0];
console.log(val_byte);

// 解析unsign int
// 由于Uint32Array的解析起点必须是4的整数倍，而在流中该数据的起点是1，所以选择先“裁剪”(slice)出要解析的流片段，再用Uint32去解析该片段
var u32buff = ab.slice(1, 5);
var u32 = new Uint32Array(u32buff);
var val_uint = u32[0];
console.log(val_uint);

// 解析unsign short
var u16buff = ab.slice(5, 7);
var u16 = new Uint16Array(u16buff);
var val_short = u16[0];
console.log(val_short);
```

## TypeArray to Array

在上文中可以看到，普通数组可以轻松的转换成TypeArray。
但TypeArray并不是Array的子集，所以它没有Array的许多方法，比如`push`
TypeArray的方法参见：[TypedArray的方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)

```javascript
var arr = [0x01,0x02,0x00,0x00,0x00,0x00,0x03];
var u8 = new Uint8Array(arr);
console.log(typeof u8.push);
```

所以需要进行转换。
TypeArray to Array的方法,在ES6中可以用Array.form实现 （[什么是Array.form](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)）

也可以比较简单的封装一下。

```javascript

function Uint8Array2Array(u8a) {
	var arr = [];
	for (var i = 0; i < u8a.length; i++) {
		arr.push(u8a[i]);
	}
	return arr;
}
```