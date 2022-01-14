# iterator

Iterator

概念

JavaScript 有四种表示“集合”的数据结构：

`Array`

、

`Object`

、

`Map`

和

`Set`



Iterator 为各种不同的数据结构提供统一的遍历机制



Iterator 的遍历过程

1. 创建一个指针对象，指向当前数据的起始位置
2. 第一次调用指针对象的`next`方法，可以将指针指向数据结构的第一个成员
3. 第二次调用指针对象的`next`方法，指针指向数据结构的第二个成员
4. 不断调用指针对象的`netx`方法，直到指向数据结构的结束位置

每一次调用`next`方法，都会返回数据结构的当前成员的信息

```纯文本
const makeIterator = array => {
	let nextIndex = 0;

	return {
		next: () => {
			return nextIndex < array.length ?
			{value: array[nextIndex++], done:false} :
			{value: undefined, done: true};
		}
	};
};
const it = makeIterator(['a', 'b']);

it.next(); // {value: 'a', done: false}
it.next(); // {value: 'b', done: false}
it.next(); // {value: undefined, done: true}
```


`next`方法返回一个对象，表示当前数据成员的信息。这个对象具有`value`和`done`两个属性，`value`属性返回当前位置的成员；`done`属性是个布尔值，表示遍历是否结束

由于 Iterator 只是把接口规范加到数据结构上，所以，遍历器与它所遍历的那个数据结构，实际上是分开的

默认 Iterator 接口

当使用

`for...of`

循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口<br />一种数据结构之遥部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”



默认的 Iterator 接口部署在数据结构的

`Symbol.itertaor`

属性

```纯文本
const obj = {
	[Symbol.iterator]: function () {
		return {
			next: () => ({value: 1, done: true})
		};
	}
};
```


原生具备 Iterator 接口的数据结构

1. Array
2. Map
3. Set
4. String
5. TypedArray
6. arguments
7. NodeList

```纯文本
const arr = ['a', 'b', 'c'];
const it = arr[Symbol.iterator]();

it.next(); // {value: 'a', done: false}
it.next(); // {value: 'b', done: false}
it.next(); // {value: 'c', done: false}
it.next(); // {value: undefined, done: true}
```


其他数据结构的 Iterator 接口，需要自己在 `Symbol.iterator` 属性上部署，这样才会被`for...of`循环遍历

```纯文本
class RangeIterator {
	constructor(start, stop) {
		this.value = start;
		this.stop = stop;
	}

	[Symbol.ierator]() { return this; }

	next() {
		let value = this.value;

		if (stop < this.stop){
			this.value++;
			return {done: false, value: value};
		}
		return {done: true, value: undefined};
	}
}
const range = new RangeIterator(0, 3);

for (const value of range){
	console.log(value);
}
// 0, 1, 2
```


对于类数组对象，部署 Iterator 接口，有一个更简便方法，就是`Symbol.iterator`方法直接引用数组的 Iterator 接口

```纯文本
NodeList.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];

NodeList.prototype[Symbol.iterator] = [][Symbol.iterator];

[...document.querySelectorAll('div')];

const it = {
	0: 'a',
	1: 'b',
	2: 'c',
	length: 3,
	[Symbol.iterator]: Array.prototype[Symbol.iterator]
};

for (const item of it){
	console.log(item);
}
// 'a', 'b', 'c'
```


调用 Iterator 接口的场合

有一些场合会默认调用 Iterator 接口

- 解构赋值

对数组和 Set 结构进行解构赋值时，会默认调用`Symbol.iterator`方法

```纯文本
const set = new Set().add('a').add('b').add('c');
const [x, y] = set; // x = 'a'; y = 'b'
const [first, ...rest] = set; // first = 'a'; rest = ['b', 'c']
```


- 扩展运算符

扩展运算符（`...`）也会调用默认的 Iterator 接口

```纯文本
const str = 'hello';
[...str]; // ['h', 'e', 'l', 'l', 'o']

const arr = ['b', 'c'];
['a', ...b, 'd']; // ['a', 'b', 'c', 'd']
```


- yield*

`yield*`后面跟的是一个可遍历的结构，他会调用该结构的遍历器接口

```纯文本
const generator = function* () {
	yield 1;
	yield* [2, 3, 4];
	yield 5;
};
const it = generator();

it.next(); // {value: 1, done: false}
it.next(); // {value: 2, done: false}
it.next(); // {value: 3, done: false}
it.next(); // {value: 4, done: false}
it.next(); // {value: 5, done: false}
it.next(); // {value: undefined, done: true}
```


- 其他场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口

`for...of`



`Array.from()`



`Map()`

, 

`Set()`

, 

`WeakMap()`

, 

`WeakSet()`



`Promise.all()`



`Promise.race()`

字符串的 Iterator 接口

字符串是一个类数组对象，也原生具有 Iterator 接口

```纯文本
const str = 'hi';
const it = str[Symbol.iterator]();

it.next(); // {value: 'h', done: false}
it.next(); // {value: 'i', done: false}
it.next(); // {value: undefined, done: true}
```


Iterator 接口和 Generator 函数

`Symbol.iterator`方法的最简单实现，就是使用 Generator 函数

```纯文本
const obj = {
	* [Symbol.iterator]() {
		yield 'hello';
		yield 'world';
	}
};

for (const x of obj){
	console.log(x);
}
// "hello"
// "world"
```


遍历器对象的return(), throw()

遍历器对象除了具有`next`方法，还可以具有`return`方法和`throw`方法

`return`方法的使用场景是，如果`for...of`循环提前退出（通常是因为出错，或者有`break`语句），就会调用`return`方法

```纯文本
function readLineSync(file) {
	return {
		[Symbol.iterator]() {
			return {
				next() {
					return {done: false};
				},
				return() {
					file.close();
					return {done: true};
				}
			};
		}
	};
}

// 情况一
for (const line of readLineSync(fileName)){
	console.log(line);
	break;
}

// 情况二
for (const line of readLineSync(fileName)){
	console.log(line);
	throw new Error();
}
```


for...of

遍历所有数据结构的统一方法



一个数据结构只要部署了

`Symbol.iterator`

属性，就被视为具有 Iterator 接口，就可以使用

`for...of`

循环遍历它的成员



`for...of`

循环可以使用的范围包括数组，Set 和 Map 结构，类数组对象，Generator 对象，以及字符串

数组

数组原生具备`iterator`接口

```纯文本
const arr = ['red', 'green', 'blue'];

for (const v of arr){
	console.log(v); // red green blue
}

const obj = {};

obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for (const v of obj){
	console.log(v); // red green blue
}
```


Set 和 Map 结构

Set 和 Map 结构也原生具有 Iterator 接口

```纯文本
const engines = new Set(['Gecko', 'Trident', 'Webkit', 'webkit']);

for (const e of engines){
	console.log(e);
}
// Gecko Trident Webkit

const es6 = new Map();

es6.set('edition', 6);
es6.set('committee', 'TC39');
es6.set('standard', 'ECMA-262');
for (const [name, value] of es6){
	console.log(`${name}: ${value}`);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```


计算生成的数据结构

`keys()`, `values()`, `entries()`都返回一个遍历器对象

```纯文本
const arr = ['a', 'b', 'c'];

for (const pair of arr.entries()){
	console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']
```


类数组对象

字符串，NodeList 对象， arguemnts 对象都属于类数组对象

```纯文本
const str = 'hello';
for (const s of str){
	console.log(s); // h e l l o
}

const paras = document.querySelectorAll('p');
for (const p of paras){
	p.classList.add('test');
}

function printArgs() {
	for (const x of arguments){
		console.log(x);
	}
}
printArgs('a', 'b'); // 'a' 'b'
```


