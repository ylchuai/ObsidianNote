# async

async 函数

含义

ES2017 标准引入了 async 函数，使得异步操作变得更加方便



它就是 Generator 函数的语法糖

```纯文本
const fs = require('fs');
const readFile = fileName => {
	return new Promise((resolve, reject) => {
		fs.readFile(fileName, (err, data) => {
			if (err){
				return reject(err);
			}
			resolve(data);
		});
	});
};
const asyncReadFile = async function () {
	const f1 = await readFile('/etc/fstab');
	const f2 = await readFile('/etc/shells');

	console.log(f1.toString());
	console.log(f2.toString());
};
```


`async`函数就是将 Generator 函数的`*`替换成`async`，将`yield`替换成`await`

`async`函数特点

1. 内置执行器

Generator 函数必须靠执行器，所以才有了`co`模块，而`async`函数自带执行器

1. 更好的语义

`async`和`await`，比起`*`和`yield`，语义更清楚了

1. 更广的适用性

`co`模块规定，`yield`命令后面只能是 Thunk 函数或者 Promise 对象，而`await`命令后面，可以是 Promise 对象和原始类型的值

1. 返回值是 Promise

`async`函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了

基本语法

`async`

函数返回一个 Promise 对象，可以使用

`then`

方法添加回调函数



当函数执行的时候，一旦遇到

`await`

就会先返回，等到异步操作完成，再接着执行函数体内后面的语句

```纯文本
async function price(name) {
	const symbol = await getStockSymbol(name);
	const stockPrice = await getStockPrice(symbol);

	return stockPrice;
}
price('goog')
	.then(result => console.log(result));

function timeout(ms) {
	return new Promise(resolve => {
		setTimeout(resolve, ms);
	});
}
async function asyncPrint(value, ms) {
	await timeout(ms);
	console.log(value);
}
asyncPrint('hello world', 50);
```


由于`async`函数返回的是 Promise 对象，可以作为`await`命令的参数

```纯文本
async function timeout(ms) {
	await new Promise(resolve => setTimeout(resolve, ms));
}
async function asyncPrint(value, ms) {
	await timeout(ms);
}
```


async 函数有多种使用方式

```纯文本
// 函数声明
async function foo() {}

// 函数表达式
const foo = async function () {};

// 对象的方法
const obj = {
	async foo() {}
};

// class 方法
class Storage {
	constructor() {
		this.cachePromise = caches.open('avatars');
	}

	async getAvatar(name) {
		const cache = await this.cachePromise;

		return cache.match(`/avatars/${name}.jpg`);
	}
}

// 箭头函数
const foo = async () => {};
```


语法

返回 Promise 对象

`async`

函数返回一个 Promise 对象



`async`

函数内部

`this`

语句的返回值，会成为

`then`

方法回调函数的参数

```纯文本
async function f() {
	return 'hello world';
}
f().then(v => console.log(v));
// 'hello world'
```


`async`函数内部抛出错误，会导致返回的 Promise 对象变为`reject`状态

```纯文本
async function f() {
	throw new Error('failed');
}
f().then(
	v => console.log(v),
	e => console.oog(e)
);
```


Promise 对象的状态变化

`async`

函数返回的 Promise 对象，必须等到内部所有

`await`

命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到

`return`

语句或者抛出错误



也就是说，只有

`async`

函数内部的异步操作执行完，才会执行

`then`

方法指定的回调函数

```纯文本
async function getTitle(url) {
	const response = await fetch(url);
	const html = await response.text();

	return html.match(/<title>([\s\S]+)<\/title>/)[1];
}
getTitle(url).then(t => console.log(t));
// 'ECMAScript 2017'
```


await

正常情况下，

`awiat`

命令后面是一个 Promise 对象，返回该对象的结果



如果不是，就直接返回对应的值

```纯文本
async function f() {
	return await 123;
}
f()
	.then(v => console.log(v));
// 123
```


JavaScript 一直没有休眠的语法。但是借助`await`命令就可以让程序停顿指定的时间

```纯文本
function sleep(interval) {
	return new Promise(resolve => setTimeout(resolve, interval));
}
async function intervalAsync() {
	for (let i = 1; i <= 5; i++){
		console.log(i);
		await sleep(1000);
	}
}
intervalAsync();
```


`await`命令后面的 Promise 对象如果变为`reject`状态，则`reject`的参数会被`catch`方法的回调函数接收到

```纯文本
async function f() {
	await Promise.reject('failed');
}
f()
	.then(v => console.log(v))
	.catch(e => console.log(e));
// failed
```


任何一个`await`语句后面的 Promise 对象变为`reject`状态，整个`async`函数都会中断执行

```纯文本
async function f() {
	await Promise.reject('failed');
	await Promise.resolve('hello world'); // 不会执行
}
```


如果内部 Promise 捕获了错误，则不会影响后面操作

```纯文本
async function f() {
	await Promise.reject('failed').catch(e => console.log(e));
	return await Promise.resolve('hello world');
}

f()
	.then(v => console.log(v));
// failed
// hello world
```


使用注意点

多个`await`命令后面的异步操作，如果不存在继发关系，最好让它们同时触发

```纯文本
// 继发关系
const foo = await getFoo();
const bar = await getBar();

// 并发1
const [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 并发2
const fooPromise = getFoo();
const barPromise = getBar();
const foo = await fooPromise;
const bar = await barPromise;
```


async 函数的实现原理

async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里

```纯文本
async function fn(args) {}
// 等同于
function fn(args) {
	return spawn(function* () {});
}
```


所有的`async`函数都可以写成上面的第二种形式，其中的`spawn`就是自动执行器

```纯文本
function spawn(gen) {
	return new Promise((resolve, reject) => {
		const g = gen();
		const step = nextF => {
			let next;

			try {
				next = nextF();
			} catech(e) {
				return reject(e);
			}
			if (next.done){
				return resolve(next.value);
			}
			Promise
				.resolve(next.vlaue)
				.then(
					v => () => gen.next(v),
					e => () => gen.throw(e)
				);
		};

		step(() => g.next(undefined));
	});
}
```


与其他异步处理方法的比较

假定某个 DOM 元素上面，部署了一系列的动画，前一个动画结束，才能开始后一个。如果当中有一个动画出错，就不再往下执行，返回上一个成功执行的动画的返回值

```纯文本
// Promise
function chainAnimationsPromise(elem, animations) {
	let ret = null;
	let p = Promise.resolve();

	for (let anim of animations){
		p = p.then(v => {
			ret = val;
			return anim(elem);
		});
	}
	return p
		.catch(e => {})
		.then(() => ret);
}

// generator
function chainAnimationsGenerator(elem, animations) {
	return spawn(function* () {
		let ret = null;
		
		try {
			for (let anim of animations) {
				ret = yield anim(elem);
			}
		} catch(e) {}
		return ret
	});
}

// async
async function chainAnimationsAsync(elem, animations) {
	let ret = null;

	try {
		for(let anim of animations) {
			ret = await anim(elem);
		}
	} catch(e) {}
	return ret;
}
```


按顺序完成异步操作

```纯文本
// Promise
function logInOrder(urls) {
	const tetPromises = urls.map(url => fetch(url).then(r => r.text()));

	textPromises
		.reduce((total, current) => {
			return total
				.then(() => current)
				.then(t => console.log(t));
		}, Promise.resolve());
}

// async
async function logInOrder(urls) {
	for (const url of urls){
		const res = await fetch(url);

		console.log(await res.text());
	}
}

// async 并发
async function logInOrder(urls) {
	// 并发读取URL
	const textPromises = urls.map(async url => {
		const res = await fetch(url);

		return res.text();
	});

	// 按次序输出
	for (const textPromise of textPromises){
		console.log(await textPromise);
	}
}
```


