# function

函数的扩展

函数参数的默认值

ES6 之前，不能直接为函数的参数指定默认值，只能采用变通方法

```纯文本
function log(x, y) {
	y = y || 'World';

	console.log(x, y)
}

log('Hello'); // Hello World
log('Hello', 'China'); // Hello China
log('Hello', ''); // Hello World
```


ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面

```纯文本
function log(x, y = 'World') {
	console.log(x, y);
}

log('Hello'); // Hello World
log('Hello', 'China'); // Hello China
log('Hello', ''); // Hello
```


可以看到，ES6 的写法比 ES5 的写法简便许多，而且非常自然

```纯文本
function Point(x = 0, y = 0) {
	this.x = x;
	this.y = y;
}

const p = new Point();

p; // { x: 0, y: 0 }
```


参数变量是默认声明的，所以不能再次声明

```纯文本
function foo(x = 5) {
	let x = 1; // SyntaxError
	const x = 2; // SyntaxError
}
```


使用默认参数时，函数不能有同名参数

```纯文本
// 不报错
function foo(x, x, y) {}

// 报错
function foo(x, x, y = 5) {}
// SyntaxError: Duplicate parameter name not allowed in this context
```


参数默认值时惰性求值的

```纯文本
let x = 99;
function foo(p = x + 1) {
	console.log(p);
}

foo(); // 100
x = 100;
foo(); // 101
```


与解构赋值默认值结合使用

参数默认值可以与解构赋值的默认值，结合起来使用

```纯文本
function foo({x, y = 5}) {
	console.log(x, y);
}

foo({}); // undefined 5
foo({x: 1}); // 1 5
foo({x: 1, y: 2}); // 1 2
foo(); // TypeError: Cannot read property 'x' of undefined
```


上面代码只使用了对象的解构赋值默认值。如果不传参，就会报错。通过提供函数参数的默认值，可以避免这种情况

```纯文本
function foo({x, y = 5} = {}) {
	console.log(x, y);
}

foo(); // undefined 5
```


下面时另一个解构赋值默认值的例子

```纯文本
function fetch(url, {body = '', method = 'GET', headers = {}} = {}) {
	console.log(method);
}

fetch('http://example.com', {}); // 'GET'
fetch('http://example.com'); // 'GET'
```


函数参数默认值和解构赋值默认值的区别

```纯文本
function m1({x = 0, y = 0} = {}) {
	return [x, y];
}
function m2({x, y} = {x: 0, y: 0}) {
	return [x, y];
}

m1(); // [0, 0]
m2(); // [0, 0]

m1({x: 3, y: 8}); // [3, 8]
m2({x: 3, y: 8}); // [3, 8]

m1({x: 3}); // [3, 0]
m2({x: 3}); // [3, undefined]

m1({}); // [0, 0]
m2({}); // [undefined, undefined]

m1({z: 3}); // [0, 0]
m2({z: 3}); // [undefined, undefined]
```


参数默认值的位置

通常情况，定义了默认值的参数，应该是函数的尾参数。如果非尾部的参数设置默认值，实际上这个参数是没发省略的

```纯文本
function f(x = 1, y) {
	return [x, y];
}

f(); // [1, undefined]
f(2); // [2, undefined]
f(, 1) // 报错
f(undefined, 1); // [1, 1]
```


如果传入`undefined`，将触发该参数等于默认值

```纯文本
function foo(x = 5, y = 6) {
	console.log(x, y);
}

foo(undefined, null); // 5 null
```


函数的 length 属性

指定了默认值以后，函数的`length`属性，将返回没有指定默认值的参数个数

```纯文本
(function (a) {}).length; // 1
(function (a = 5) {}).length; // 0
(function (a, b, c = 5) {}).length; // 2
```


> 这是因为`length`属性的含义是，该函数预期传入的参数个数


同理，rest 参数也不会计入`length`属性

```纯文本
(function (..arg) {}).length; // 0
```


如果设置了默认值的参数不是尾参数，那么`length`属性也不再计入后面的参数了

```纯文本
(function (a = 0, b, c) {}).length; // 0
(function (a, b = 1, c) {}).length; // 1
```


作用域

一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的

```纯文本
let x = 1;
function f(x, y = x) {
	console.log(y);
}

f(2); // 2
```


应用

利用参数默认值，可以指定某一个参数不得省略

```纯文本
function throwIfMissing() {
	throw new Error('Missing parameter');
}
function foo(mustBeProvided = throwIfMissing()) {
	return mustBeProvided;
}

foo(); // Error: Missing parameter
```


rest 参数

ES6 引入 rest 参数，用于获取函数的多余参数，这样就不需要使用`arguments`对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中

```纯文本
function add(...values) {
	let sum = 0;

	for (let val of values) {
		sum += val;
	}

	return sum;
}

add(2, 5, 3); // 10
```


下面是一个 rest 参数代替 arguments 变量的例子

```纯文本
function sortNumbers() {
	return Array.prototype.slice.call(arguments).sort();
}

const sortNumbers = (...numbers) => numbers.sort();
```


rest 参数之后不能再有其他参数

```纯文本
function f(a, ...b, c) {}
// SyntaxError: Rest parameter must be last formal parameter
```


函数的`length`属性，不包括 rest 参数

```纯文本
(function (a) {}).length; // 1
(function (...a) {}).length; // 0
(function (a, ...b) {}).length; // 1
```


严格模式

ES5 中，函数内部可以设定为严格模式

```纯文本
function doSomething() {
	'use strict';
}
```


ES2016 规定，只要函数参数适用了默认值、解构赋值、扩展运算符，函数内部就不能显式设定为严格模式

```纯文本
// 报错
function doSomething(a, b = a) {
	'use strict';
}

// 报错
const doSomething = function ({a, b}) {
	'use strict';
}

// 报错
const doSomething = (...a) => {
	'use strict';
}

const obj = {
	// 报错
	doSomething({a, b}) {
		'use strict';
	}
}
```


name 属性

函数的`name`属性，返回该函数的函数名

```纯文本
function foo() {}
```


> 这个属性早就被浏览器广泛支持，但是直到 ES6，才将其写入标准


如果将一个匿名函数赋值给一个变量，ES5 中`name`返回空字符串，ES6 中返回实际的函数名

```纯文本
var f = function () {}

// ES5
f.name; // ""
// ES6
f.name; // "f"
```


如果将一个具名函数赋值给一个变量，ES5 和 ES6 的`name`属性都返回这个具名函数原本的名字

```纯文本
const bar = function baz() {}

// ES5
bar.name; // "baz"
// ES6
bar.name; // "baz"
```


`Function`构造函数返回的函数实例，`name`属性的值为`anonymous`

```纯文本
(new Function).name; // "anonymous"
```


`bind`返回的函数，`name`属性会加上`bound`前缀

```纯文本
function foo() {}
foo.bind({}).name; // "bound foo"

(function () {}).bind({}).name; // "bound "
```


箭头函数

ES6 允许使用“箭头”（`=>`）定义函数

```纯文本
const f = v => v;

// 等同于
var f = function (v) {
	return v;
}
```


如果箭头函数不需要参数或者需要多个参数，就使用一个圆括号代表参数

```纯文本
const f = () => 5;
const sum = (num1, num2) => num1 + num2;
```


如果箭头函数的函数体多于一条语句，就要使用大括号把它们括起来，并且使用`return`语句返回

```纯文本
const sum = (num1, num2) => {
	return num1 + num2;
}
```


如果要直接返回一个对象，必须在对象外面加上括号

```纯文本
const getTempItem = id => ({id: id, name: 'Temp'});
```


箭头函数可以与变量结构结合使用

```纯文本
const full = ({first, last}) => first + ' ' + last;
```


箭头函数使表达更加简洁

```纯文本
const isEven = n => n % 2 === 0;
const square = n => n * n;
```


箭头函数的一个用处是简化回调函数

```纯文本
// 正常函数写法
[1, 2, 3].map(function (x) {
	return x * x;	
});

// 箭头函数写法
[1, 2, 3].map(x => x * x);

// 正常函数写法
var result = values.sort(function (a, b) {
	return a - b;
});		

// 箭头函数写法
const result = values.sort((a, b) => a - b);
```


与 rest 参数结合使用

```纯文本
const numbers = (...nums) => nums;

numbers(1, 2, 3); // [1, 2, 3]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3); // [1, [2, 3]]
```


箭头函数有几个使用注意点

1. 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象
2. 不可当作构造函数
3. 不可使用`arguments`对象
4. 不可使用`yield`命令

第一点尤其重要，`this`对象的指向是可变的，但在箭头函数中，它是固定的

```纯文本
function foo() {
	setTimeout(() => {
		console.log(`id: ${this.id}`)	
	}, 100);
}
let id = 21;

foo.call({id: 42}); // id: 42
```


> `this`指向的固定化，并不是因为箭头函数内部有绑定`this`的机制，实际原因是箭头函数根本没有自己的`this`，导致内部的`this`就是外层代码块的`this`。正是因为它没有`this`，所以也就不能用做构造函数


所以，箭头函数转成 ES5 的代码如下

```纯文本
// ES6
function foo() {
	setTimeout(() => {
		console.log(`id: ${this.id}`);
	}, 100);
}

// ES5
function foo() {
	var _this = this;

	setTimeout(function () {
		console.log('id: ' + _this.id);
	}, 100);
}
```


除了`this`，以下三个变量在箭头函数之中也不存在：`arguments`、`suoer`、`new.target`

不适用场景

第一个场景是定义对象的方法，且该方法内部包括`this`

```纯文本
const cat = {
	lives: 9,
	jumps: () => this.lives
}

cat.jumps(); // undefined
```


> 上面箭头函数使得`this`指向全局对象。这是因为对象不构成单独的作用域，导致`jumps`箭头函数定义时的作用域就是全局作用域


第二个场景是需要动态`this`的时候，也不应该使用箭头函数

```纯文本
const button = document.getElementById('press');

button.addEventListener('click', () => {
	this.classList.toogle('on');
});
```


嵌套的箭头函数

箭头函数的内部，还可以再使用箭头函数

```纯文本
// ES5
function insert(value) {
	return {into: function (array) {
		return {after: function (afterValue) {
			array.splice(array.indexOf(afterValue) + 1, 0, value);
			return array;
		}}	
	}}
}

// ES6
const insert = value => ({into: array => ({after: afterValue => {
	array.splice(array.indexOf(afterValue) + 1, 0, value);
	return array;
}})});

insert(2).into([1, 3]).after(1); // [1, 2, 3]
```


尾调用优化

尾调用

尾调用（Tail Call）是函数式编程的一个重要概念，就是指某个函数的最后一步是调用另一个函数

```纯文本
function f(x) {
	return g(x);
}
```


以下三种情况，都不属于尾调用

```纯文本
// 情况一
function f(x) {
	let y = g(x);

	return y;
}

// 情况二
function f(x) {
	return g(x) + 1;
}

// 情况三
function f(x) {
	g(x);
}
```


尾调用不一定要出现在函数尾部，只要是最后一步操作即可

```纯文本
function f(x) {
	if (x > 0) {
		return m(x);
	}
	return n(x);
}
```


> 上面，函数`m`和`n`都属于尾调用


尾调用优化

尾调用之所以与其他调用不同，就在于它的特殊的调用位置。

我们知道，函数调用会在内存形成一个“调用记录”，又称“调用帧”，保存调用位置和内部变量等信息。

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置，内部变量等信息都不会再用到了，只要调用内层函数的调用帧，取代外层函数的调用帧就可以了。

```纯文本
function f() {
	let m = 1;
	let n = 2;

	return g(m + n);
}
f();

// 等同于
function f() {
	return g(3);
}
f();

// 等同于
g(3);
```


这就叫作“尾调用优化”，即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次调用时，调用帧只有一项，这将大大节省内存，这就是“尾调用优化”的意义

> 注意，只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧


尾递归

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

递归非常消耗内存，因为需要同时保存成千上百个调用帧，但对尾递归来说，只存在一个调用帧。

```纯文本
function factorial(n) {
	if (n === 1) {
		return 1;
	}
	return n * factorial(n - 1);
}
factorial(5); // 120
```


上面代码是一个阶乘函数，计算`n`的阶乘，最多要保存`n`个调用记录，复杂度 O(n)

如果改成尾递归，只保留一个调用记录，复杂度 O(1)

```纯文本
function factorial(n, total) {
	if (n === 1) {
		return total;
	}
	return factorial(n - 1, n * total);
}
factorial(5, 1)
```


递归函数的改写

尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。两个方法可以解决这个问题。

方法一：在尾递归函数之外，再提供一个正常函数

```纯文本
function tailFactorial(n, total) {
	if (n === 1) {
		return total;
	}
	return tailFactorial(n - 1, n * total);
}
function factorial(n) {
	return tailFactorial(n, 1);
}
factorial(5); // 120
```


方法二：采用 ES6 的函数默认值

```纯文本
function factorial(n, total = 1) {
	if (n === 1) {
		return total;
	}
	return factorial(n - 1, n * total);
}
factorial(5); // 120
```


函数参数的尾逗号

ES2017 允许函数的最后一个参数有尾逗号

```纯文本
function clownsEverywhere(
	param1,
	param2,
) {}
clownsEverywhere(
	'foo',
	'bar',
);
```


Function.prototype.toString

ES2019 对函数实例的`toString()`方法做出了修改。`toString()`方法返回函数代码本身，以前会省略注释和空格。修改后，返回一摸一样的原始代码

```纯文本
function /* foo comment */ foo() {}
foo.toString();
// "function /* foo comment*/ foo() {}"
```


catch 命令的参数修改

JavaScript 语言的`try...catch`结构，以前明确要求`catch`后面必须跟参数。ES2019 做出了改变，允许`catch`语句省略参数

```纯文本
try {

} catch {
	
}
```


