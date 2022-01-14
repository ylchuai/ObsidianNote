# destructuring

变量的解构赋值

数组的解构赋值

基础用法

ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构

```纯文本
let [a, b, c] = [1, 2, 3]
```


本质上，这种写法属于**模式匹配**，只要等号两边的模式相同，左边的变量就会被赋予对应的值

```纯文本
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ['foo', 'bar', 'baz'];
third // "baz"

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```


> 如果解构不成功，变量的值就等于`undefined`


另一种情况是不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。这种情况下，解构依然可以成功

```纯文本
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```


> 如果等号的右边不是数组（严格说，不是可遍历的解构，即没有部署`Iterator`接口），那么将会报错


对于`Set`结构，也可以使用数组的解构赋值

```纯文本
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
y // "b"
z // "c"
```


事实上，只要某种数据结构居右`Iterator`接口，都可以采用数组形式的解构赋值

```纯文本
function* fibs(){
	let a = 0;
	let b = 1;
	while (true){
		yield a;
		[a, b] = [b, a + b]
	}
}
let [first, second, third, fourth, fifth, sixth] = fibs();
first // 0
sixth // 5
```


> 解构赋值会依次从`Iterator`获取值


默认值

解构赋值允许指定默认值

```纯文本
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a'];
x // "a"
y // "b"

let [first, second = 1] = [0, undefined];
first // 0
second // 1
```


ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以只有当一个数组成员严格等于`undefined`，默认值才会生效

```纯文本
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```


如果默认值是一个表达式，那么这个表达式是惰性求值，即只有在用到的时候，才会求值

```纯文本
const f = () => 0
let [x = f()] = [1]
// f 不会执行
```


默认值可以引用解构赋值的其他变量，但该变量必须已经声明

```纯文本
let [x = 1, y = x] = [];
x // 1
y // 1

let [x = 1, y = x] = [2];
x // 2
y // 2

let [x = 1, y = x] = [1, 2];
x // 1
y // 2
```


对象的解构赋值

基础用法

解构不仅可以用于数组，还可以用于对象

```纯文本
let { foo, bar } = { foo: 'aaa', bar: 'bbb' }
foo // "aaa"
bar // "bbb"
```


对象的解构与数组有一个重要的不同，数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值

```纯文本
let { bar, foo, baz } = { foo: 'a', bar: 'b' }
foo // 'a'
bar // 'b'
baz // undefined
```


> 如果解构失败，变量的值等于`undefined`


对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量

```纯文本
let { sin, cos } = Math;

const { log } = console;
log('hello') // hello
```


如果变量名与属性名不一致，必须写成下面这样

```纯文本
let { foo: baz } = { foo: 'a', bar: 'b' }
baz // "a"
```


> 也就是说，对象的解构赋值的内部机制，是先找到同名属性，然后再赋值给对应的变量。真正被赋值的是后者，而不是前者


与数组一样，解构也可以用于嵌套结构的对象

```纯文本
let obj = {
	p: [
		'Hello',
		{ y: 'world' }
	]
}
let { p: [x, { y }]} = obj;
x // 'Hello'
y // 'World'
```


注意，这时`p`是一个模式，不是变量，因此不会被赋值。如果`p`也要作为变量赋值，可以写成下面这样

```纯文本
let obj = {
	p: [
		'Hello',
		{ y: 'World' }
	]
}
let { p, p: [x, { y }]} = obj;
x // "Hello"
y // "World"
p // ["Hello", {y: "World"}]
```


下面是另一个例子

```纯文本
const node = {
	loc: {
		start: {
			line: 1,
			column: 5
		}
	}
}
let { loc, loc: { start }, loc: { start: { line} } } = node;
line // 1
loc // {start: {line: 1, column: 5}}
start // {line: 1, column: 5}
```


下面是嵌套赋值的例子

```纯文本
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });
obj // {prop: 123}
arr // [true]
```


注意，对象的解构可以取到继承的属性

```纯文本
const obj1 = {}
const obj2 = { foo: 'bar' }

Object.setPrototypeOf(obj1, obj2);

const { foo } = obj1;
foo // "bar"
```


默认值

对象的解构也可以指定默认值

```纯文本
const { x = 3 } = {};
x // 3

const { x, y = 5 } = { x: 1 };
x // 1
y // 5

const { x: y = 3 } = {};
y // 3

const { x: y = 3 } = { x: 5 };
y // 5

const { message: msg = 'Something went wrong' } = {}
msg // "Something went wrong"
```


注意

1. 如果要将一个已经声明的变量用于解构赋值，要将整个解构赋值语句，放在一个圆括号里面

```纯文本
let x;
({ x } = { x: 1 })
```


1. 解构赋值允许等号左边的模式之中，可以不妨值任何变量名

```纯文本
({} = [true, false]);
({} = 'abc');
({} = []);
```


1. 由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构

```纯文本
let arr = [1, 2, 3];
let { 0: first, [arr.length - 1]: last } = arr;
first // 1
last // 3
```


字符串的解构赋值

字符串也可以解构赋值。这是因为此时，字符串被转换成一个类似数组的对象

```纯文本
const [a, b, c, d, e] = 'hello';
a // "a"
e // "e"
```


类似数组的对象都有一个`length`属性，因此可以对整个属性解构赋值

```纯文本
let { length: len } = 'hello';
len // 5
```


数值和布尔值的解构赋值

解构赋值时，如果等号右边是数值和布尔值，则会先转为对象

```纯文本
let { toString: s } = 123;
s === Number.prototype.toString; // true

let { toString: s } = true;
s === Boolean.prototype.toString
```


**解构赋值的规则是，只要等号左边的值不是对象或者数组，就先将其转为对象。由于**`**undefined**`**和**`**null**`**无法转为对象，所以对他们进行解构赋值，都会报错**

```纯文本
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```


函数参数的解构赋值

函数的参数也可以使用解构赋值

```纯文本
function add([x, y]){
	return x + y;
}
add([1, 2])

[[1, 2], [3, 4]].map(([a, b]) => a + b); // [3, 7]
```


函数参数的解构赋值也可以使用默认值

```纯文本
function move({x = 0, y = 0} = {}){
	return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]

function move({x, y} = {x: 0, y: 0}){
	return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```


用途

- 交换变量的值

```纯文本
let x = 1;
let y = 2;

[x, y] = [y, x];
```


- 从函数返回多个值

```纯文本
function example(){
	return [1, 2, 3];
}
let [a, b, c] = example();

function example(){
	return {
		foo: 1,
		bar: 2
	}
}
let { foo, bar } = example();
```


- 函数参数的定义

```纯文本
function f([x, y, z]){}
f([1, 2, 3])

function f({x, y, z}){}
f({z: 3, y: 2, x: 1})
```


- 提取 JSON 数据

```纯文本
let jsonData = {
	id: 42,
	status: 'ok',
	data: [867, 5309]
}
let { id, status, data: number } = jsonData;
id // 42
status // "ok"
number // [867, 5309]
```


- 函数参数的默认值

```纯文本
jQuery.ajax = function (url, {
	async = true,
	before
} = {}){}
```


- 遍历 Map 结构

```纯文本
const map = new Map();

map.set('first', 'hello');
map.set('second', 'world');
for (let [key, value] of map){
	console.log(key + ' is ' + value);
}
// first is hello
// second is world
```


- 输入模块的指定方法

```纯文本
const { SourceMapConsumer, SourceNode } = require('source-map')
```


