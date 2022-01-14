# generator

Generator 函数

简介

基本概念

Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同



语法上，首先可以把它理解成“状态机”，封装了多个内部状态



执行 Generator 函数返回一个遍历器对象，可以依次遍历 Generator 函数内部的每一个状态



形式上，Generator 函数是一个普通函数，但有两个特点。一是，

`function`

关键字和函数名之间有一个星号；二是，函数内部使用

`yield`

表达式，定义不同的内部状态

```纯文本
function* gen() {
	yield 'hello';
	yield 'world';
	return 'ending';
}
const it = gen();
```


然后，Generator 函数的调用方式与普通函数一样。不同之处是，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象

```纯文本
it.next(); // {value: 'hello', done: false}
it.next(); // {value: 'world', done: false}
it.next(); // {value: 'ending', done: false}
it.next(); // {value: undefined, done: true}
```


调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历对象的`next`方法，就会返回一个有着`value`和`done`属性的对象。`value`属性表示当前内部状态的值，也就是`yield`表达式后面的值；`done`属性是一个布尔值，表示是否遍历结束

yield

由于 Generator 函数返回的遍历器对象，只有调用`next`方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数，`yield`表达式就是暂停标志

遍历器对象的`next`方法的运行逻辑

1. 遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回对象的`value`属性值
2. 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式
3. 如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面表达式的值，作为返回对象的`value`属性值
4. 如果该函数没有`return`语句，则返回对象的`value`属性值为`undefined`

注意，`yield`表达式后面的表达式，只有当调用`next`方法，内部指针指向该语句时才会执行

```纯文本
function* gen() {
	yield 123 + 456;
}
```


Generator 函数可以不用`yield`表达式，变成一个单纯的暂缓执行函数

```纯文本
function* f() {
	console.log('done');
}
const it = f();

setTimeout(() => generator.next(), 2000);
// 两秒后打印'done'
```


与 Iterator 接口的关系

任何一个对象的

`Symbol.iterator`

方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象



由于 Generator 函数就时遍历器生成函数，因此可以把 Generator 赋值给对象的

`Symbol.iterator`

属性，从而使该对象具有 Iterator 接口

```纯文本
const obj = {};

obj[Symbol.iterator] = function* () {
	yield 1;
	yield 2;
	yield 3;
};
[...obj]; // [1, 2, 3]
```


Generator 函数执行后，返回一个遍历器对象。该对象本身也具有`Symbol.iterator`属性，执行后返回自身

```纯文本
function* gen() {}
const g = gen();

g[Symbol.iterator]() === g; // true
```


next 方法的参数

`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值

```纯文本
function* f() {
	for (let i = 0; true; i++){
		const reset = yield i;

		if (reset){
			i = -1;
		}
	}
}
const g = f();

g.next(); // {value: 0, done: false}
g.next(); // {value: 1, done: false}
g.next(true); // {value: 0, done: false}
```


这个功能有很重要的语法意义。Generator 函数从暂停状态到恢复运行，它的上下文状态是不变的



通过

`next`

方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内注入值



也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为

```纯文本
function* foo(x) {
	const y = 2 * (yield (x + 1));
	const z = yield (y / 3);

	return (x + y + z);
}
const a = foo(5);

a.next(); // {value: 6, done: false}
a.next(); // {value: NaN, done: false}
a.next(); // {value: NaN, done: true}

const b = foo(5);

b.next(); // {value: 6, done: false}
b.next(12); // {value: 8, done: false}
b.next(13); // {value: 42, done: true}
```


注意，由于`next`方法的参数表示上一个`yield`表达式的返回值，所以在第一次调用`next`方法时，传递参数时无效的

```纯文本
function* gen() {
	console.log('Started');
	console.log(`1. ${yield}`);
	console.log(`2. ${yield}`);
	return 'result';
}
const it = gen();

it.next(); // Started
it.next('a'); // 1. a
it.next('b'); // 2. b
```


for...of

`for...of`循环可以自动遍历 Generator 函数运行时生成的 Iterator 对象，此时不再需要调用`next`方法

```纯文本
function* foo() {
	yield 1;
	yield 2;
	yield 3;
	return 4;
}

for (const v of foo()){
	console.log(v);
}
// 1 2 3
```


注意，一旦`next`方法返回对象的`done`属性为`true`，`for...of`循环就会终止，且不包含该返回对象

```纯文本
function* fibonacci() {
	let [prev, curr] = [0, 1];

	for (;;){
		yield curr;
		[prev, curr] = [curr, prev + curr];
	}
}
for (const n of fibonacci()){
	if (n > 1000){
		break;
	}
	console.log(n);
}
```


利用`for...of`循环，可以写出遍历任意对象的方法

```纯文本
function* objEntries(obj) {
	const propKeys = Reflect.ownKeys(obj);

	for (const p of propKeys) {
		yield [p, obj[p]];
	}
}
const jane = {first: 'Jane', last: 'Doe'};

for (const [key, value] of objEntries(jane)){
	console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```


除了`for..of`循环外，扩展运算符（`...`），解构赋值和`Array.from()`方法内部调用的，都是遍历器接口。这意味着，它们都可以将 Generator 函数返回的 Iterator 对象，作为参数

```纯文本
function* numbers() {
	yield 1;
	yield 2;
	return 3;
	yield 4;
}

[...numbers()]; // [1, 2]
Array.from(numbers()); // [1, 2]

const [x, y] = numbers();
x; // 1
y; // 2

for (const n of numbers()) {
	console.log(n);
}
// 1
// 2
```


Generator.prototype.throw()

Generator 函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获

```纯文本
const g = function* () {
	try {
		yield;
	} catch(e) {
		console.log('inner', e);
	}
}
const i = g();

g.next();
try {
	i.throw('a');
	i.throw('b');
} catch(e) {
	console.log('outter', e);
}
// inner a
// outter b
```


`throw`方法可以接受一个参数，该参数会被`catch`语句接受

```纯文本
const g = function* () {
	try {
		yield;
	} catch(e) {
		console.log(e);
	}
};
const i = g();

i.next();
i.throw(new Error('failed'));
// Error: failed
```


如果 Generator 函数内部没有部署`tray...catch`，那么`throw`方法抛出的错误，将被外部`try...catch`捕获

```纯文本
const g = function* () {
	while(true) {
		yield;
	}
};
const i = g();

i.next();
try {
	i.throw('a');
	i.throw('b');
} catch(e) {
	console.log('outter', e);
}
// outter a
```


如果 Generator 函数内部和外部，都么有部署`try...catch`，那么程序将报错中断执行

```纯文本
const gen = function* () {
	yield console.log('hello');
	yield console.log('world');
};
const g = gen();

g.next();
g.throw();
// hello
// Uncaught undefined
```


`throw`方法抛出的错误要被内部捕获，前提是必须至少执行一次`next`方法

```纯文本
function* gen() {
	try {
		yield 1;
	} catch(e) {
		console.log('inner');
	}
}
const g = gen();

g.throw(1);
// Uncaught 1
```


> 因为，第一次执行`next`方法，等同于启动执行 Generator 函数的内部代码


`throw`方法捕获以后，会附带执行下一条`yield`表达式。也就是说，会附带执行一次`next`方法

```纯文本
const gen = function* () {
	try {
		yield console.log('a');
	} catch(e) {}
	yield console.log('b');
	yield console.log('c');
};
const g = gen();

g.next(); // a
g.throw(); // b
g.next(); // c
```


> 只要 Generator 函数内部部署了`try...catch`，那么遍历器的`throw`方法抛出的错误，不影响下一次遍历


Generator 函数体外抛出的错误，可以在函数体内捕获。反过来，Generator 函数体内抛出的错误，也可以被函数体外的`catch`捕获

```纯文本
function* foo() {
	const x = yield 3;
	const y = x.toUpperCase();
	yield y;
}
const it = foo();

it.next();
try {
	it.next(42);
} catch(err) {
	console.log(err);
}
```


一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就不再执行下去。如果此后还调用`next`方法，将返回`{value: undefined, done: true}`对象

```纯文本
function* g() {
  yield 1;
  console.log('throwing an exception');
  throw new Error('generator broke!');
  yield 2;
  yield 3;
}

function log(generator) {
  var v;
  console.log('starting generator');
  try {
    v = generator.next();
    console.log('第一次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第二次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第三次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  console.log('caller done');
}

log(g());
// starting generator
// 第一次运行next方法 { value: 1, done: false }
// throwing an exception
// 捕捉错误 { value: 1, done: false }
// 第三次运行next方法 { value: undefined, done: true }
// caller done
```


Generator.prototype.return()

Generator 函数返回的遍历器对象，还有一个`return`方法，可以返回给定的值，并且终结遍历 Generator 函数

```纯文本
function* gen() {
	yield 1;
	yield 2;
	yield 3;
}
const g = gen();

g.next(); // {vlaue: 1, done: false}
g.return('foo'); // {value: 'foo', done: true}
g.next(); // {vlaue: undefined, done: true}
```


如果，`return`方法调用时，不提供参数，则返回值的`value`属性为`undefined`

```纯文本
function* gen() {
	yield 1;
	yield 2;
	yield 3;
}
const g = gen();

g.next(); // {value: 1, done: false}
g.return(); // {value: undefined, done: true}
```


如果，Generator 函数内部有`try...finally`，且正在执行`try`代码块，那么`return`方法会导致立刻进入`finally`代码块，执行完以后，整个函数才会结束

```纯文本
function* numbers() {
	yield 1;
	try {
		yield 2;
		yield 3;
	} finally {
		yield 4;
		yield 5;
	}
	yield 6;
}
const g = numbers();

g.next(); // {value: 1, done: false}
g.next(); // {value: 2, done: false}
g.return(7); // {value: 4, done: false}
g.next(); {value: 5, done: false}
g.next(); // {value: 7, done: true}
```


next(), throw(), return()的共同点

`next`, `throw`, `return`这三个方法本质上是用一件事。它们的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换`yield`表达式

`next()`是将`yield`表达式替换成一个值

```纯文本
const g = function* (x, y) {
	const result = yield x + y;
	return result;
};
const gen = g(1, 2);

gen.next(); // {value: 3, done:false}
gen.next(1); // {value: 1, done: true}
// 相当于 const result = yield x + y
// 替换成 const result = 1
```


`throw()`是将`yield`表达式替换成一个`throw`语句

```纯文本
gen.throw(new Error('failed')); // Uncaught Error: failed
// 相当于将 const result = yield x + y
// 替换成 const result = throw(new Error('failed'))
```


`return()`是将`yield`表达式替换成一个`return`语句

```纯文本
gen.return(2); // {value: 2, done: true}
// 相当于 const result = yield x + y
// 替换成 const result = return 2
```


yield*

如果在 Generator 函数内部，调用另一个 Generator 函数，需要在前者的函数体内部，自己手动完成遍历

```纯文本
function* foo() {
	yield 'a';
	yield 'b';
}
function* bar() {
	yield 'x';
	for (const i of foo()){
		console.log(i);
	}
	yield 'y';
}

for (const v of bar()) {
	console.log(v);
}
// x
// a
// b
// y
```


ES6 提供了`yield*`表达式，作为解决办法，用来在一个 Generator 函数里面执行另一个 Generator 函数

```纯文本
function* bar() {
	yield 'x';
	yield* foo();
	yield 'y';
}
// 等同于
function* bar() {
	yield 'x';
	yield 'a';
	yield 'b';
	yield 'y';
}
// 等同于
function* bar() {
	yield 'x';
	for (const v of foo()){
		yield v;
	}
	yield 'y';
}

for (const v of bar()){
	console.log(v);
}
// 'x'
// 'a'
// 'b'
// 'y'
```


实际上，任何数据结构只要有 Iterator 接口，就可以被`yield*`遍历

```纯文本
const read = (function* (){
	yield 'hello'	;
	yield* 'hello';
})();

read.next().value; // 'hello'
read.next().value; // 'h'
```


如果被代理的 Generator 函数有`return`语句，那么就可以向代理它的 Generator 函数返回数据

```纯文本
function* foo() {
	yield 2;
	yield 3;
	return 'foo';
}
function* bar() {
	yield 1;
	const v = yield* foo();

	console.log(`v: ${v}`);
	yield 4;
}
const it = bar();

it.next(); // {value: 1, done: false}
it.next(); // {value: 2, done: false}
it.next(); // {value: 3, done: false}
it.next();
// 'v: foo'
// {value: 4, done: false}
it.next(); // {value: undefined, done: true}
```


`yield*`命令可以很方便地取出嵌套数组的所有成员

```纯文本
function* iterTree(tree) {
	if (Array.isArray(tree)){
		for (let i = 0; i < tree.length; i++){
			yield* iterTree(tree[i]);
		}
	} else {
		yield tree;
	}
}
const tree = ['a', ['b', 'c'], ['d', 'e']];

[...iterTree(tree)] // ["a", "b", "c", "d", "e"]
```


作为对象属性的 Generator 函数

如果一个对象的属性是 Generator 函数，可以简写为

```纯文本
const obj = {
	* gen() {}
};
// 等同于
const obj = {
	gen: function* () {}
};
```


Generator 函数的this

Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数`prototype`对象上的方法

```纯文本
function* g() {}
g.prototype.hello = function () {
	return 'hi';
};
const obj = g();

obj instanceof g; // true
obj.hello(); // 'hi'
```


如果，把 Gnerator 函数当作构造函数，并不会生效，因为 Generator 函数总是返回遍历器对象，而不是`this`对象

```纯文本
function* g() {
	this.a = 11;
}
const obj = g();

obj.next();
obj.a; // undefined
```


