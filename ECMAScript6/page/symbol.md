# symbol

Symbol

概述

ES6 引入了一种新的原始数据类型`Symbol`，表示独一无二的值

> 七种数据类型：`null`, `undefined`, `number`, `string`, `boolean`, `object`, `symbol`


Symbol 值通过`Symbol`函数生成。也就是说对象的属性名现在可以有两种类型（`string`, `Symbol`）

```纯文本
const s = Symbol();

typeof s; // "symbol"
```


> `Symbol`函数前不能使用`new`，因为`Symbol`不是个构造函数


`Symbol`函数可以接受一个字符串作为参数，表示对`Symbol`实例的描述，主要是为了在控制台显示，或者转化为字符串时，比较容易区分

```纯文本
const s1 = Symbol('foo');

f1; // Symbol(foo)
s1.toString(); // "Symbol(foo)"
```


如果 Symbol 的参数是一个对象，就会调用该对象的`toString`方法，将其转化为字符串，然后才生成一个 Symbol 值

```纯文本
const obj = {toString() {return 'abc'}};
const sym = Symbol(obj);

sym; // Symbol(abc)
```


`Symbol`函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的`Symbol`函数的返回值时不相等的

```纯文本
let s1 = Symbol();
let s2 = Symbol();
s1 === s2; // false

let s1 = Symbol('foo');
let s2 = Symbol('foo');
s1 === s2; // false
```


Symbol 值不能与其他类型的值进行计算

```纯文本
let sym = Symbol('my Symbol');

'your Symbol is' + sym; // TypeError: can't convert symbol to string
```


但是，Symbol 值可以显式转为字符串

```纯文本
let sym = Symbol('My Symbol');

String(sym); // "Symbol(abc)"
sym.toString(); // "Symbol(abc)"
```


另外，Symbol 值也可以转化为布尔值，但不能转为数值

```纯文本
let sym = Symbol();

Boolean(sym); // true
!sym; // false

if (sym) {}
Number(sym); // TypeError
sym + 2; // TypeError
```


Symbol.prototype.description

返回 Symbol 值的描述

```纯文本
const sym = Symbol('foo');

sym.description; // "foo"
```


作为属性名的 Symbol

由于每一个 Symbol 值都是不相等的，这意味着 Symbol 值可以作为标识符，用于对象的属性名，就能保证不会出现相同的属性

```纯文本
let sym = Symbol();

let a = {};
a[sym] = 'hello';

let a = {
	[sym]: 'hello'
};

let a = {};
Object.defineProperty(a, sym, {value: 'hello'});
```


注意，Symbol 值作为对象属性时，不能用点运算符

```纯文本
const sym = Symbol();
const a = {};

a.sym = 'hello';
a[sym]; // undefined
a['sym']; // "hello"
```


Symbol 类型还可以用于定义一组常量，保证这组常量的值都不相等

```纯文本
const log = {};

log.levels = {
	DEBUG: Symbol('debug'),
	INFO: Symbol('info'),
	WARN: Symbol('warn')
};
console.log(log.levels.DEBUG, 'debug message');
console.log(log.levels.INFO, 'info message');
```


消除魔术字符串

魔术字符串指的是，在代码之中多次出现，与代码形成强耦合的某一个具体的字符串或数值

```纯文本
function getArea(shape, options) {
	let area = 0;

	switch(shape) {
		case 'Triangle': // 魔术字符串
			area = .5 * options.width * options.height;
			break;
	}
	return area;
}
getArea('Triangle', {width: 100, height: 100});
```


常用的消除魔术字符串的方法，就是把它写成一个变量

```纯文本
const shapeType = {triangle: 'Triangle'};
function getArea(shape, options) {
	let area = 0;

	switch(shape) {
		case shapeType.triangle:
			area = .5 * options.height * options.width;
			break;
	}
	return area;
}
getArea(shapeType.triangle, {width: 100, height: 100});
```


如果仔细分析，可以发现`shapeType.triangle`等于哪个值并不重要，只要确保不会跟其他`shapeType`属性的值冲突即可。因此，这里就很适合该用`Symbol`值

```纯文本
const shapeType = {triangle: Symbol()};
```


属性名的遍历

Symbol 作为属性名，遍历对象的时候，该属性不会出现在`for...in`, `for...of`循环中，也不会被`Object.keys()`, `Object.getOwnPropertyNames()`, `JSON.stringify()`返回

但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols`方法，可以获取指定对象的所有 Symbol 属性名

```纯文本
const obj = {};

let a = Symbol('a');
let b = Symbol('b');

obj[a] = 'hello';
obj[b] = 'world';

Object.getOwnPropertySymbols(obj); // [Symbol(a), Symbol(b)]
```


`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名

```纯文本
let obj = {
	[Symbol('foo')]: 1,
	enum: 2,
	nonEnum: 3
};

Reflect.ownKeys(obj); // ["enum", "nonEnum", Symbol(foo)]
```


由于以 Symbol 值作为键名，不会被常规遍历方法得到。我们可以利用这个属性，为对象定义一些非私有的、但又希望只用于内部的方法

```纯文本
let size = Symbol('size');
class Collection {
	constructor() {
		this[size] = 0;
	}

	add(item) {
		this[this[size]] = item;
		this[size]++;
	}

	static sizeOf(instance) {
		return instance[size];
	}
}
let x = new Collection();

Collection.sizeOf(x); // 0
x.add('foo');
Collection.sizeOf(x); // 1

Object.keys(x); ['0']
Object.getOwnPropertyNames(x); // ['0']
Object.getOwnPropertySymbols(x); // [Symbol(size)]
```


Symbol.for(), Symbol.keyFor()

有时，我们希望重新使用同一个 Symbol 值，`Symbol.for`方法就可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到全局

```纯文本
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2; // true
```


`Symbol.for()`与`Symbol()`这两种写法，都会生成新的 Symbol 值。它们的区别是，前者会被登记到全局环境中供搜索，后者不会

```纯文本
Symbol.for('bar') === Symbol.for('bar'); // true
Symbol('bar') === Symbol('bar'); // false
```


`Symbol.keyFor`方法返回一个已登记的 Symbol 类型值的`key`

```纯文本
let s1 = Symbol.for('foo');
Symbol.keyFor(s1); // "foo"

let s2 = Symbol('foo');
Symbol.keyFor(s2); // undefined
```


注意，`Symbol.for()`为 Symbol 值登记的名字，是全局环境的，不管有没有在全局环境运行

```纯文本
function foo() {
	return Symbol.for('bar');
}
const x = foo();
const y = Symbol.for('bar');

x === y; // true
```


`Symbol.for()`的这个全局登记特性，可以用在不同的 iframe 或 service worker 中取到同一个值

```纯文本
const iframe = docuemnt.createElement('iframe');
iframe.src = String(window.location);
docuemnt.body.appendChild(iframe);

iframe.contentWindow.Symbol.for('for') === Symbol.for('foo');
```


模块的 Singleton 模式

Singleton 模式指的是调用一个类，任何时候返回的都是同一个实例

对于 Node 来说，模块文件可以看成一个类。怎么保证每次执行这个模块文件，返回的都是同一个实例

很容易想到，可以把实例放到顶层对象`gloabl`

```纯文本
// mod.js
function A() {
	this.foo = 'hello';
}

if (!global._foo){
	global._foo = new A();
}

module.exports = global._foo;
```


然后，加载上面的`mod.js`

```纯文本
const a = require('./mod.js');

console.log(a.foo); // "hello"
```


上面代码，变量`a`任何时候加载都是`A`的同一实例

但是，这里有一个问题，全局变量`global._foo`是可写的，任何文件都可以修改

```纯文本
global._foo = {foo: 'world'};
const a = require('./mod.js');

console.log(a.foo); // "world"
```


为了防止这种情况出现，我们就可以使用 Symbol

```纯文本
// mod.js
const FOO_KEY = Symbol('foo');
function A() {
	this.foo = 'hello';
}

if (!global[FOO_KEY]) {
	global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```


内置的 Symbol 值

除了定义自己使用的 Symbol 值外，ES6 还提供了 11 个内置的 Symbol 值，指向语言内部使用的方法

- Symbol.hasInstance

判断是否为该对象的实例，会调用`foo instanceof Foo`，在语言内部实际上调用的是`Foo[Symbol.hasInstance](foo)`

- Symbol.isConcatSpreadable

一个布尔值，表示该对象用于`Array.prototype.concat()`时，是否可以展开

- Symbol.species

指向一个构造函数，创建衍生对象时，会使用该属性

- Symbol.match

指向一个函数，当执行`str.match()`时，如果该属性存在，会调用它，返回该方法的返回值

- Symbol.replace

指向一个方法，当该对象被`String.prototype.replace()`调用时，会返回该方法的返回值

- Symbol.search

指向一个方法，当该对象被`String.prototype.search()`调用时，会返回该方法的返回值

- Symbol.split

指向一个方法，当该对象被`String.prototype.split()`调用时，返回该方法的返回值

- Symbol.iterator

指向该对象默认遍历器方法

- Symbol.toPrimitive

指向一个方法，该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值

- Symbol.toStringTag

指向一个方法，该对象被`Object.prototype.toString()`调用时，如果这个属性存在，它的返回值会出现在`toString`方法返回的字符串中

- Symbol.unscopables

指向一个对象，该对象指定了使用`width`关键字时，哪些属性会被`width`环境排除

