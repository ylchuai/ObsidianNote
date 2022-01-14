# variable

变量声明

let

ES6 新增了 `let` 命令，用来声明变量。

```纯文本
{
	let a = 10;
	var b = 1;
}
a // ReferenceError: a is not defined
b // 1
```


只在声明所在的作用域内有效

```纯文本
for (let i = 0; i < 10; i += 1){}
i // ReferenceError: i is not defined

for (var i = 0; i < 10; i += 1){}
i // 10
```


> `for` 循环还有个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域


不存在变量提升

变量不可在声明之前使用

```纯文本
foo // undefined
var foo

bar // ReferenceError: Cannot access 'd' before initialization
let bar
```


暂时性死区

只要块级作用域内存在`let`命令，它所声明的变量就绑定这个区域，不再受外部的影响

```纯文本
let tmp = 123;

if (true){
	tmp = 321; // ReferenceError: Cannot access 'tmp' before initialization
	let tmp;
}
```


> ES6 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，葱一开始就形成了封闭作用域。凡是在声明之前使用这些变量，就会报错


`typeof`不再是一个百分之百安全的操作

```纯文本
typeof x; // ReferenceError: Cannot access 'x' before initialization
let x;

typeof y; // "undefined"
```


有些**死区**比较隐蔽，不太容易发现

```纯文本
let bar = (x = y, y = 2) => [x, y]
bar // ReferenceError: Cannot access 'y' before initialization

let baz = (x = 2, y = x) => [x, y]
baz() // [2, 2]
```


总之，暂时性死区的本质就是，只要一进入当前作用域，所有使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

不允许重复声明

`let`不允许在相同作用域内，重复声明同一个变量

```纯文本
function func(){
	let a = 10;
	let a = 1;
}
// SyntaxError: Identifier 'a' has already been declared

function fn(arg){
	let arg;
}
// SyntaxError: Identifier 'arg' has already been declared

function f(arg){
	{
		let arg;
	}
}
// 正确
```


块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景

```纯文本
var tmp = new Date()

function f(){
	console.log(tmp);
	if (false){
		var tmp = 'hello wolrd'
	}
}
f(); // undefined
```


> 原因在于变量提升，导致内层的`tmp`变量覆盖了外层的`tmp`变量


用来计数的循环变量泄露为全局变量

```纯文本
for (var i = 0; i < 3; i += 1){}
i // 3
```


ES6 的块级作用域

`let` 实际上为 JavaScript 新增了块级作用域

```纯文本
let n = 5;
function f1(){
	if (true){
		let n = 10
	}
	console.log(n); // 5
}
```


ES6 允许块级作用域的任意嵌套

```纯文本
{{{{
	{let insane = 'Hello World'}
	console.log(insane); // 报错
}}}};
```


块级作用域与函数声明

```纯文本
function f() { console.log('I am outside!'); }
function fn() {
	if (false) {
		// 重复声明一次函数f
		function f() { console.log('I am inside!'); }
	}
	f();
}
fn() // f is not a function
```


如果改变了块级作用域内声明的函数处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 在规定，浏览器实现可以不遵守上面的规定。

- 允许在块级作用域内声明函数
- 函数声明类似`var`, 即会提升到全局作用域或函数作用域的头部
- 同时，函数声明还会提升到所在块级作用域的头部

> 考虑到环境导致的行为差异太大，应该避免块级作用域内声明函数。如果确实需要，也应该写出函数表达式，而不是函数声明语句


ES6 的块级作用域必须有大括号

```纯文本
if (true) let x= 1;
// SyntaxError: Lexical declaration cannot appear in a single-statement context
```


const

`const` 声明一个只读的常量。一旦声明，常量的值就不能改变

```纯文本
const PI = 3.1415;

PI // 3.1415
PI = 3 // TypeError: Assignment to constant variable.
```


`const` 声明的变量不得改变值，这意味着，`const` 一旦声明变量，就必须立即初始化，不能留到以后赋值。

```纯文本
const foo;
// SyntaxError: Missing initializer in const declaration
```


只在声明所在的作用域内有效

```纯文本
if (true){
	const MAX = 5;
}
MAX // ReferenceError: MAX is not defined
```


不允许重复声明

```纯文本
var message = 'Hello!';
let age = 25;

const message = 'Goodbye!'
// SyntaxError: Identifier 'message' has already been declared
const age = 30;
// SyntaxError: Identifier 'age' has already been declared
```


`**const**`** 实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，**`**const**`** 只能保证这个指针是固定的（即总是指向另外一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明作为常量必须非常小心**

```纯文本
const foo = {};

foo.prop = 123;
console.log(foo.prop); // 123

foo = {} // TypeError: Assignment to constant variable.
```


> 上面代码中，常量`foo`存储的是一个地址，这个地址指向一个对象。不可变的只是这个地址，即不能把`foo`指向另一个地址，但对象本身是可变的，所以依然可以为其添加新的属性。


冻结对象

```纯文本
const foo = Object.freeze({});

foo.prop = 123;
console.log(foo); // {}
```


彻底冻结函数

```纯文本
const constantize = obj => {
	Object.freeze(obj);
	Object.keys(obj).forEach(k => {
		if (typeof obj[k] === 'object'){
			constantize(obj[k])
		}
	})
}
```


ES6 变量的6种声明方式

```纯文本
var
function
let
const
import
class
```


顶层对象

ES5 中，顶层对象的属性与全局变量是等价的

```纯文本
window.a = 1;
a // 1

a = 2;
window.a // 2
```


> 顶层对象的属性与全局变量挂钩，被认为是 JavaScript 语言最大的设计败笔之一。


```纯文本
1、没发在编译时就报出变量未声明的错误		
2、容易创建全局变量		
3、顶层对象的属性是到处可以读写的，这非常不利于模块化编程		
4、`window` 对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的
```


ES6 为了改变这一点，一方面，为了保持兼容性，`var`和`function`命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，`let`、`const`和`class`命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐渐与顶层对象的属性脱钩。

```纯文本
var a = 1;
window.a // 1

let b = 1;
window.b // undefined
```


globalThis 对象

JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都在这个环境中运行。但是，顶层对象在各个实现里面是不统一的。

- 浏览器里面，顶层对象是`window`，但是 Node 和 Web Worker 没有 `window`
- 浏览器和 Web Worker 里面，`self`也指向顶层对象，但是 Node 没有 `self`
- Node 里面，顶层对象是`global`，但其他环境都不支持

ES6 引入`globalThis`作为顶层对象。也就是说，任何环境下，`globalThis`都是存在的，都可以从它拿到顶层对象，指向全局环境下的`this`

