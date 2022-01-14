# arrayBuffer

ArrayBuffer

`ArrayBuffer`对象、`TypedArray`视图和`DataView`试图是 JavaScript 操作二进制数据的一个接口<br />它们都是以数组的语法处理二进制数据，所以统称为二进制数组

这个接口的原始设计目的，与 WebGL 项目有关。<br />所谓 WebGL，就是指浏览器与显卡之间的通信接口，为了满足 JavaScript 与显卡之间大量的、实时的数据交换，它们之间的数据通信必须是二进制的，而不能是传统的文本格式

二进制数组允许开发者以数组下标的形式，直接操作内存，大大增强了 JavaScript 处理二进制数据的能力，使得开发者有可能通过 JavaScript 与操作系统的原生接口进行二进制通信

二进制数组由三类对象构成：

1. `ArrayBuffer`对象：代表内存之中的一段二进制数据，可以通过“视图”进行操作
2. `TypedArray`视图：共包含 9 种类型的视图
3. `DataView`视图：可以自定义复合格式的视图

`ArrayBuffer`对象代表原始的二进制数据<br />`TypedArray`视图用来读写简单类型的二进制数据<br />`DataView`视图用来读写复杂类型的二进制数据

`TypeArray`视图支持的数据类型一共有 9 种

|数据类型|字节长度|含义|
|---|---|---|
|Int8|1|8 位带符号整数|
|Uint8|1|8 位不带符号整数|
|Uint8C|1|8 位不带符号整数（自动过滤溢出）|
|Int16|2|16 位带符号整数|
|Uint16|2|16 位不带符号整数|
|Int32|4|32 位带符号整数|
|Uint32|4|32 位不带符号整数|
|Float32|4|32 位浮点数|
|Float64|8|64 位浮点数|



> 二进制数组并不是真正的数组，而是类数组对象


很多浏览器操作的 API，用到了二进制数组操作二进制数据

- Canvas
- Fetch API
- File API
- WebSockets
- XMLHttpRequest

ArrayBuffer 对象

`ArrayBuffer`对象代表存储二进制数据的一段内存<br />它不能直接读写，只能通过视图来读写，视图的作用是以指定格式读取二进制数据<br />`ArrayBuffer`也会一个构造函数，可以分配一段可以存放数据的连续内存区域

```纯文本
const buf = new ArrayBuffer(32);
```


上面代码生成了一段 32 字节的内存区域，每个字节的默认值都是 0<br />`ArrayBuffer`构造函数的参数所需要内存的大小（单位是字节）

为了读写这段内容，需要为它指定视图

```纯文本
const buf = new ArrayBuffer(32);
const dataView = new DataView(buf);

dataView.getUint8(0);
```


`TypedArray`视图，它不是一个构造函数，而是一组构造函数

```纯文本
const buffer = new ArrayBuffer(12);
const x1 = new Int32Array(buffer);
const x2 = new Uint8Array(buffer);

x1[0] = 1;
x2[0] = 2;
x1[0]; // 2
```


> 由于两个视图对应的是同一段内存，一个视图修改底层视图，会影响到另一个视图


`TypedArray`视图的构造函数，除了接受`ArrayBuffer`实例作为参数，还可以接受普通数组作为参数，直接分配内存生成底层的`ArrayBuffer`实例，并同时完成对这段内存的赋值

```纯文本
const typedArray = new Uint8Array([0, 1, 2]);

typedArray.length; // 3
typedArray[0] = 5;
typedArray; // [5, 1, 2]
```


ArrayBuffer.prototype.byteLength

返回所分配内存区域的字节长度

```纯文本
const buffer = new ArrayBuffer(32);

buffer.byteLength; // 32
```


如果要分配的内存区域很大，有可能分配失败，所以有必要检测是否分配成功

```纯文本
if (buffer.byteLength === n) {
	// succes
} else {
	// failed
}
```


ArrayBuffer.prototype.slice()

将内存区域的一部分，拷贝生成一个新的`ArrayBuffer`对象

```纯文本
const buffer = new ArrayBuffer(8);
const newBuffer = buffer.slice(0, 3);
```


`slice`方法接受两个参数，第一个参数表示拷贝开始的字节序号，第二个参数表示结束的字节序号<br />如果省略第二个参数，则默认到`ArrayBuffer`对象的结尾

除了`slice`方法，`ArrayBuffer`不提供任何直接读写内存的方法，只允许在其上方建立视图，然后通过视图读写

ArrayBuffer.isView()

返回一个布尔值，表示参数是否为`ArrayBuffer`的视图实例

```纯文本
const buffer = new ArrayBuffer(8);
ArrayBuffer.isView(buffer); // false

const v = new Int32Array(buffer);
ArrayBuffer.isView(v); // true
```


TypedArray 视图

`ArrayBuffer`对象作为内存区域，可以存放多种类型的数据<br />同一段内存，不同数据有不同的解读方式，这就叫做"视图"<br />`ArrayBuffer`有两种视图，一种是`TypedArray`视图，另一种是`DataView`视图

目前，`TyoedArray`视图有 9 种类型，每一种类型都是一个构造函数

这 9 种构造函数生成的数组，统称为`TypedArray`视图<br />它们很像普通数组，都有`length`属性，都能用`[]`获取单个元素，所有数组的方法，在它们上面都能使用

- TypedArray 数组的所有成员，都是同一种类型
- TypedArray 数组的成员是连续的，不会有空位
- TypedArray 数组成员的默认值为 0
- TypedArray 数组只是一层视图，本身不存储数据，它的数据都存储在底层的`ArrayBuffer`对象之中

构造函数

构造函数有多种用法

（1）TypedArray(buffer, byteOffset = 0, length?)

同一个`ArrayBuffer`对象上，可以根据不同的数据类型，建立多个视图

```纯文本
const b = new ArrayBuffer(8);
const v1 = new Int32Array(b);
const v2 = new Uint8Array(b, 2);
const v3 = new Int16Array(b, 2, 2);
```


视图的构造函数可以接受三个参数

- buffer（必需）：视图对应的底层`ArrayBuffer`对象
- byteOffset（可选）：视图开始的字节序号，默认从 0 开始
- length（可选）：视图包含的数据个数，默认直到本段内存结束

`byteOffset`参数必须与所建立的数据类型一致

```纯文本
const buffer = new ArrayBuffer(8);
const i16 = new Int16Array(buffer, 1);
// Error
```


> 带符号的 16 位整数需要两个字节，所以`byteOffset`参数必须能被 2 整除


（2）TypedArray(length)

视图还可以不通过`ArrayBuffer`对象，直接分配内存而生成

```纯文本
const f64a = new Float64Array(8);

f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
```


（3）TypedArray(TypedArray)

TyoedArray 数组的构造函数，可以接受另一个`TypedArray`实例作为参数

```纯文本
const typedArray = new Int8Array(new Uint8Array(4));
```


（4）TypedArray(arrayLinkeObject)

构造函数的参数也可以是一个普通数组

```纯文本
const typedArray = new Uint8Array([1, 2, 3, 4]);
```


数组方法

普通数组的操作方法和属性，对 TypedArray 数组完全适用<br />TypedArray 数组没有`concat`方法<br />如果想要合并多个 TypedArray 数组

```纯文本
function concatenate(resultConstructor, ...arrays) {
	let totalLength = 0;

	for (let arr of arrays) {
		totalLenth += arr.length;
	}
	let result = new resultConstructor(totalLength);
	let offset = 0;

	for (let arr of arrays) {
		result.set(arr, offsets);
		offset += arr.length;
	}
	return result;
}
concatenate(Uint8Array, Uint8Array.of(1, 2), Uint8Array.of(3, 4));
// Uint8Array [1, 2, 3, 4]
```


字节序

字节序指的是数值在内存中的表示方式

```纯文本
const buffer = new ArrayBuffer(16);
const int32View = new Int32Array(buffer);

for (let i = 0; i < int32View.length; i++) {
	int32View[i] = i * 2;
}
```


由于每个 32 位整数占据 4 个字节，所以一共可以写入 4 个整数

```纯文本
const int16View = new Int16Array(buffer);

for (let i = 0; i < int16View.length; i++) {
	console.log(`Entry ${i}: ${int16View[i]}`);
}
```


由于每个 16 位整数占据 2 个字节，所以整个`ArrayBuffer`对象现在分成 8 段

小端字节序将最不重要的字节排在前面<br />大端字节序将最重要的字节排在前面<br />TypedArray 数组内部也采用小端字节序读写数据<br />TypedArray 数组将无法正确解析大端字节序<br />为了解决这个问题，JavaScript 引入`DataView`对象，可以设定字节序

```纯文本
const buffer = new ArrayBuffer(4);
const v1 = new Uint8Array(buffer);

v1[0] = 2;
v1[1] = 1;
v1[2] = 3;
v1[3] = 7;

const uInt16View = new Uint16Array(buffer);

// 计算机采用小端字节序，所以前两个字节等于 258
uInt16View[0] === 258; // true
```


BYTES_PER_ELEMENT

表示这种数据类型占据的字节数

```纯文本
Int8Array.BYTES_PER_ELEMENT; // 1
Uint16Array.BYTES_PER_ELEMENT; // 2
Float64Array.BYTES_PER_ELEMENT; // 8
```


ArrayBuffer 与字符串的相互转化

`ArrayBuffer`和字符串的相互转化，使用原生`TextEncoder`和`TextDecoder`

```纯文本
function ab2str(input, outputEncoding) {
	const decoder = new TextDecoder(outputEncoding);

	return decoder.decode(input);
}

function str2ab(input) {
	const view = str2Unit8Array(input);

	return view.buffer;
}
function str2Uint8Array(input) {
	const encoder = new TextEncoder();
	const view = encoder.encode(input);

	return view;
}
```


溢出

不同的视图类型，所能容纳的数值范围是确定的<br />超出这个范围，就会出现溢出

TypedArray 数组的溢出处理规则，就是抛弃溢出的位，然后按照视图类型进行解释

```纯文本
const uint8 = new Uint8Array(1);

uint8[0] = 256;
uint8[0]; // 0

uint8[0] = -1;
uint8[0]; // 255
```


TypedArray.prototype.buffer

返回整段内存区域对应的`ArrayBuffer`对象

```纯文本
const a = new Float32Array(64);
const b = new Uint8Array(a.buffer);
```


TypedArray.prototype.byteLength

返回 TypedArray 数组占据的内存长度，单位为字节

```纯文本
const b = new ArrayBuffer(8);
const v1 = new Int32Array(b);

v1.byteLength; // 8
```


TypedArray.prototype.byteOffset

返回 TypedArray 数组从底层`ArrayBuffer`对象的起始字节序号

```纯文本
const b = new ArrayBuffer(8);
const v1 = new Int32Array(b);
const v2 = new Uint8Array(b, 2);

v1.byteOffset; // 0
v2.byteOffset; // 2
```


TypedArray.prototype.length

表示`TypedArray`数组有多少成员

```纯文本
const a = new Int16Array(8);

a.length; // 8
a.byteLength; // 16
```


TypedArray.prototype.set()

用于复制数组，也就是将一段内容完全复制到另一段内存

```纯文本
const a = new Uint8Array(8);
const b = new Uint8Array(8);

b.set(a);
```


TypedArray.prototype.subarray()

对于 TypedArray 数组的一部分，再建立一个新的视图

```纯文本
const a = new Uint16Array(8);
const b = a.subarray(2, 3);

a.byteLength; // 16
a.byteLength; // 2
```


TypedArray.prototype.slice()

返回一个指定位置的新的`TypedArray`实例

```纯文本
let ui8 = Uint8Array(0, 1, 2);

ui8.slice(-1); // Uint8Array [2]
```


TypedArray.of()

用于将参数转化成一个`TypedArray`实例

```纯文本
Float32Array.of(0.151, -8, 3.7);
// Float32Array [0.151, -8, 3.7]
```


TypedArray.from()

接受一个可遍历的数据结构作为参数，返回一个基于这个结构的`TypedArray`实例

```纯文本
Uint16Array.from([0, 1, 2]);
// Uint16Array [0, 1, 2]
```


复合视图

在同一段内存当中，可以依次存放不同类型的数据

```纯文本
const buffer = new ArrayBuffer(24);
const idView = new Uint32Array(buffer, 0, 1);
const usernameView = new Uint8Array(buffer, 4, 16);
const amountDueView = new Float32Array(buffer, 20, 1);
```


DataView 视图

如果一段数据包含多种类型，除了建立`ArrayBuffer`对象的复合视图以外，还可以通过`DataView`视图进行操作

`DataView`视图提供更多操作选项，而且支持设定字节序<br />`TypedArray`视图，用来向网卡、声卡之类的本机设备传送数据<br />`DataView`视图，用于处理网络设备传来的数据

`DataView`视图本身也是构造函数，接受一个`ArrayBuffer`对象作为参数

```纯文本
new DataView(buffer, byteOffset, length);

const buffer = new ArrayBuffer(24);
const dv = new DataView(buffer);
```


`DataView`实例有一下属性

- DataView.prototype.buffer：返回对应的 ArrayBuffer 对象
- DataView.prototype.byteLength：返回占据的内存字节长度
- DataView.prototype.byteOffset：返回当前视图对应的 ArrayBuffer 对象起始字节序号

`DataView`实例提供 8 个方法读取内存

- getInt8：读取 1 个字节，返回一个 8 位整数
- getUint8：读取 1 个字节，返回一个无符号的 8 位整数
- getInt16：读取 2 个字节，返回一个 16 位整数
- getUint16：读取 2 个字节，返回一个无符号的 16 位整数
- getInt32：读取 4 个字节，返回一个 32 位整数
- getUint32：读取 4 个字节，返回一个无符号的 32 位整数
- getFloat32：读取 4 个字节，返回一个 32 位浮点数
- getFloat64：读取 8 个字节，返回一个 64 位浮点数

这一系列`get`方法的参数都是一个字节序号，表示从哪个字节开始读取

```纯文本
const buffer = new ArrayBuffer(24);
cosnt dv = new DataView(buffer);

const v1 = dv.getUint8(0);
const v2 = dv.getUint16(1);
const v3 = dv.getUint16(3);
```


如果一次读取两个或两个以上的字节，就必须明确数据的存储方式<br />`DataView`的`get`方法默认使用大端字节序解读数据<br />使用小端字节序解读，必须在`get`方法的第二个参数指定为`true`

```纯文本
const v1 = dv.getUint16(1, true);
const v2 = dv.getUint16(3, false);
const v3 = dv.getUint16(3);
```


`DataView`实例提供 8 个方法写入内存

- setInt8：写入 1 个字节的 8 位整数
- setUint8：写入 1 个字节的 8 位无符号整数
- setInt16：写入 2 个字节的 16 位整数
- setUint16：写入 2 个字节的 16 位无符号整数
- setInt32：写入 4 个字节的 32 位整数
- setUint32：写入 4 个字节的 32 位无符号整数
- setFloat32：写入 4 个字节的 32 位浮点数
- setFloat64：写入 8 个字节的 64 位浮点数

这一系列`set`方法接受两个参数<br />第一个参数是字节序号，表示从哪个字节开始写入<br />第二个参数是写入的数据<br />写入两个或两个以上字节的方法，需要指定第三个参数，表示存储方法

```纯文本
dv.setInt32(0, 25, false);
dv.setInt32(4, 25);
dv.setFloat32(8, 2.5, true);
```


二进制数组的应用

大量的 Web API 用到了`ArrayBuffer`对象和它的视图对象

AJAX

传统上，服务器通过 AJAX 操作只能返回文本数据，即`responseType`属性默认为`text`<br />`XMLHttpRequest`第二版本允许服务器返回二进制数据<br />如果明确知道返回的二进制数据类型，可以把`responseType`设为`arraybuffer`<br />如果不明确，就设为`blob`

```纯文本
let xhr = new XMLHttpRequest();

xhr.open('GET', url);
xhr.responseType = 'arraybuffer';

xhr.onload = function () {
	let arrayBuffer = xhr.response;
};
xhr.send();
```


Canvas

网页`Canvas`元素输出的二进制像素数据，就是 TypedArray 数组

```纯文本
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const uint8ClampedArray = imageData.data;
```


WebSocket

`WebSocket`可以通过`ArrayBuffer`，发送或者接受二进制数据

```纯文本
let socket = new WebSocket('ws://127.0.0.1:8081');

socket.binaryType = 'arraybuffer';
socket.addEventListener('oepn', function (event) {
	const typedArray = new Uint8Array(4);

	socket.send(typedArray.buffer);
});

socket.addEventListener('message', function (event) {
	const arrayBuffer = event.data;	
});
```


Fetch API

Fetch API 取回的数据，就是`ArrayBuffer`对象

```纯文本
fetch(url)
	.then(res => res.arrayBuffer())
	.then(arrayBuffer => {
		// ...
	});
```


File API

如果知道一个文件的二进制数据类型，也可以将这文件读取为`ArrayBuffer`对象

```纯文本
const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];
const reader = new FileReader();

reader.readAsArrayBuffer(file);
reader.onload = function () {
	const arrayBuffer = reader.result;

	// ...
}
```


