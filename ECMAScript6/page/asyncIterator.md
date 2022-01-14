# asyncIterator

异步遍历器

同步遍历器的问题

Iterator 接口是一种数据遍历的协议，只要调用遍历器对象的`next`方法，就会得到一个对象，表示当前遍历指针所在位置的信息

```纯文本
function idMaker() {
	let index = 0;

	return {
		next: function () {
			return {
				value: index++,
				done: false
			}
		}
	};
}
const it = idMaker();

it.next().value; // 0
it.next().value; // 1
it.next().value; // 2
```


`next`方法必须是同步的，只要调用就必须立刻返回值

```纯文本
function idMaker() {
	let index = 0;

	return {
		next: function () {
			return new Promise((s, j) => {
				setTimeout(() => s({
					value: index++,
					done: false
				}), 1000);
			});
		}
	};
}
```


目前的解决方案是，将异步操作包装成 Thunk 函数或者 Promise 对象

```纯文本
function idMaker() {
	let index = 0;

	return {
		next: () => {
			return {
				value: new Promise(s => setTimeout(() => s(index++)), 1000),
				done: false
			};
		}
	};
}
const it = idMaker();

it.next().value.then(d => console.log(d)); // 1
it.next().value.then(d => console.log(d)); // 2
it.next().value.then(d => console.log(d)); // 3
```


异步遍历器接口

异步遍历器最大的特点，就是调用遍历器的`next`方法，返回一个 Promise 对象

```纯文本
asyncIterator
	.next()
	.then(({value, done}) => console.log(value, done));
```


同步遍历器的接口，部署在`Symbol.iterator`属性上<br />异步遍历器的接口，部署在`Symbol.asyncIterator`属性上<br />只要对象的`Symbol.asyncIterator`属性有值，就应该对它进行异步遍历

```纯文本
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();

asyncIterator
	.next()
	.then(r => {
		console.log(r); // {value: 'a', done: false}
	})
	.then(r => {
		console.log(r); // {value: 'b', done: false}	
	})
	.then(r => {
		console.log(r); // {value: undefined, done: true}	
	})
```


由于异步遍历器的`next`方法，返回的是一个 Promise 对象，所以可以把它放在`await`后面

```纯文本
async function f() {
	const asyncIterable = createAsyncIterable(['a', 'b']);
	const asyncIterator = asyncIterable[Symbol.asyncIterator]();

	console.log(await asyncIterator.next()); // {value: 'a', done: false}
	console.log(await asyncIterator.next()); // {value: 'b', done: false}
	console.log(await asyncIterator.next()); // {value: undefined, done: true}
}
```


异步遍历器的`next`方法可以连续调用，不必等到上一次产生的 Promise 对象`resolve`以后再调用

```纯文本
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();
const [{value: v1}, {value: v2}] = await Promise.all([
	asyncIterator.next(),
	asyncIterator.next()
]);

console.log(v1, v2); // a b
```


for await...of

`for...of`

用于遍历同步的 Iterator 接口



`for await...of`

用于遍历异步的 Iterator 接口

```纯文本
for await (const x of createAsyncIterable(['a', 'b'])){
	console.log(x);
}
// a
// b
```


`for await...of`也可以遍历同步 Iterator 接口

```纯文本
for await (const x of ['a', 'b']){
	console.log(x);
}
// a
// b
```


Node v10支持异步遍历器，Stream 就部署了这个接口

```纯文本
// 传统写法
function main(inputFilePath) {
	const readStream = fs.createReadStream(
		inputFilePath,
		{encoding: 'utf8', higWaterMark: 1024}
	);

	readStream.on('data', chunk => {
		console.log(`>>> ${chunk}`);
	});
	readStream.on('end', () => {
		console.log('### DONE ###');
	});
}

// 异步遍历器写法
async function main(inputFilePath) {
	const readStream = fs.createReadStream(
		inputFilePath,
		{encoding: 'utf8', higWaterMark: 1024}
	);

	for await (const chunk of readStream){
		console.log(`>>> ${chunk}`);
	}
	console.log('### DONE ###');
}
```


异步 Generator 函数

异步 Generator 函数的作用，是返回一个异步遍历器对象<br />异步 Generator 函数就是`async`函数与 Generator 函数的结合

```纯文本
async function* gen() {
	yield 'hello';
}
const genObj = gen();

genObj.next().then(x => console.log(x));
// {value: 'hello', done: false}
```


异步遍历器的设计目的之一，就是 Generator 函数处理同步操作和异步操作时，能够使用同一套接口

```纯文本
// 同步 Generator 函数
function* map(iterable, func) {
	const iter = iterable[Symbol.iterator]();

	while(true) {
		const {value, done} = iter.next();

		if (done){
			break;
		}
		yield func(value);
	}
}

// 异步 Generator 函数
async function* map(iterable, func) {
	const iter = iterable[Symbol.asyncIterator]();

	while(true) {
		const {value, done} = await iter.next();

		if (done) {
			break;
		}
		yield func(value);
	}
}
```


异步 Generator 函数内部，能够同时使用`yield`和`await`命令

```纯文本
async function* readLines(path) {
	let file = await fileOpen(pat);

	try {
		while(!file.EOF) {
			yield await file.readLine();
		}
	} finally {
		await file.close();
	}
}

(async function() {
	for await (const line of readLines(filePath))	{
		console.log(line);
	}
})();
```


yield* 语句

yield* 也可以跟一个异步遍历器

```纯文本
async function* gen1() {
	yield 'a';
	yield 'b';
	return 2;
}
async function* gen2() {
	const result = yield* gen1();
}
```


与同步 Generator 函数一样，`for await...of`会展开`yield*`

```纯文本
(async function () {
	for await (const x of gen2())	{
		console.log(x);
	}
})();
// a
// b
```


