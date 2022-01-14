# setmap

Set 和 Map 数据结构

Set

基本用法

ES6 提供了新的数据结构 Set。它类似于数组，但成员的值都是唯一的，没有重复的值

`Set`本身是一个构造函数，用来生成 Set 数据结构

```纯文本
const s = new Set();
const arr = [2, 3, 5, 4, 5, 2, 2];

arr.forEach(x => s.add(x));
for (let i of s){
	console.log(i);
}
// 2 3 5 4
```


`set`函数可以接受一个数组（或具有 iterator 接口的其他数据结构）作为参数，用来初始化

```纯文本
const set = new Set([1, 2, 3, 4, 5]);
[...set]; // [1, 2, 3, 4]

const items = new Set([1, 2, 3, 4, 5, 5, 5]);
items.size; // 5

const set = new Set(document.querySelectorAll('div'));
set.size; // 13

const set = new Set();
document
	.querySelectorAll('div')
	.forEach(div => set.add(div));
set.size; // 13
```


数组去重的方法

```纯文本
[...new Set(array)];
```


也可以去除字符串里重复的字符

```纯文本
[...new Set('abbc')].join(''); // "abc"
```


向 Set 加入值的时候，不会发生类型转化。Set 内部判断两个值是否相等，使用的算法叫“Same-vaule-zero equality”，类似于精确相等运算符（`===`），主要的区别在于`NaN`等于自身

```纯文本
const set = new Set();

set.add(NaN);
set.add(NaN);
set; // Set {NaN}
```


另外，两个对象总是不相等

```纯文本
const set = new Set();

set.add({});
set.add({});
set.size; // 2
```


Set 实例的属性和方法

- Set.prototype.constructor：构造函数
- Set.prototype.size：返回`set`实例的成员总数
- Set.prototype.add(value)：添加值，返回 Set 结构本身
- Set.prototype.delete(value)：删除值，返回成功与否的布尔值
- Set.prototype.has(value)：返回一个布尔值，表示该值是否为`set`的成员
- Set.prototype.clear()：清空所有值，没有返回值

```纯文本
const set = new Set();

set.add(1).add(2).add(2);
set.size; // 2
set.has(1); // true
set.delete(2); // true
set.clear();
set.size; // 0
```


`Array.from()`可以将 Set 结构转化为数组

```纯文本
const set = new Set([1, 2, 3]);
const arr = Array.from(set);

arr; // [1, 2, 3]
```


去除数组重复成员的另一个方法

```纯文本
const dedupe = arr => Array.from(new Set(arr));

dedupe([1, 2, 2, 3]); // [1, 2, 3]
```


遍历方法

Set 结构的实例有四个遍历方法

- set.prototype.keys()：键名遍历器
- set.prototype.values()：键值遍历器
- set.prototype.entries()：键值对遍历器
- set.prototype.forEach()：回调函数遍历每个成员

`keys`方法、`values`方法和`entries`方法返回的都是遍历器对象。由于 Set 结构没有键名，所以`keys`方法和`values`方法行为一致

```纯文本
const set = new set(['red', 'green', 'blue']);

for (const k of set.keys()){
	console.log(k);
}
// red green blue

for (const v of set.values()){
	console.log(v);
}
// red green blue

for (const i of set.entries()){
	console.log(i);
}
// ["red", "red"] ["green", "green"] ["blue", "blue"]
```


Set 结构的实例默认默认可遍历，它默认值的遍历器生成函数就是它的`values`方法

```纯文本
Set.prototype[Symbol.iterator] === Set.prototype.values; // true
```


这就意味着，可以省略`values`方法，直接使用`for...of`遍历 Set

```纯文本
const set = new Set(['red', 'green', 'blue']);

for (const v of set){
	console.log(v);
}
// red green blue
```


`forEach`方法的参数就是一个处理函数，该函数的参数依次为键值、键名、解构本身

```纯文本
const set = new Set([1, 4, 9]);

set.forEach((v, k) => console.log(`${k}: ${v}`));
// 1: 1 4: 4 9: 9
```


Set 可以很容易实现并集、交集、差集

```纯文本
const a = new Set([1, 2, 3]);
const b = new Set([4, 3, 2]);

// 并集
new Set([...a, ...b]); // Set {1, 2, 3, 4}
// 交集
new Set([...a].filter(x => b.has(x))); // Set {2, 3}
// 差集
new Set([...a].filter(x => !b.has(x))); // Set {1}
```


WeakSet

基本用法

WeakSet 结构与 Set 类似，也是不重复值的集合。它与 Set 有两个区别

1. WeakSet 结构的成员只能是对象
2. WeakSet 中的对象都是弱引用（垃圾回收机制不考虑 WeakSet 对该对象的引用）

```纯文本
const ws = new WeakSet();

ws.add(1); // TypeError: Invalid value used in weak set
ws.add(Symbol()); // TypeError: Invalid value used in weak set
```


WeakSet 是一个构造函数，可以使用`new`命令，创建 WeakSet 数据结构

```纯文本
const ws = new WeakSet();
```


作为构造函数，WeakSet 可以接受一个数组或者类数组对象（任何具有 Iterator 接口的对象）作为参数

```纯文本
const a = [[1, 2], [3, 4]];
new WeakSet(a); // WeakSet {[1, 2], [3, 4]}

const a = [3, 4];
new WeakSet(a); // TypeError: Invalid value used in weak set
```


WeakSet 实例的属性和方法

- WeakSet.prototype.constructor：构造函数
- WeakSet.prototype.add(value)：添加值，返回 WeakSet 结构本身
- WeakSet.prototype.delete(value)：删除值，返回成功与否的布尔值
- WeakSet.prototype.has(value)：返回一个布尔值，表示该值是否为`WeakSet`的成员

```纯文本
const ws = new WeakSet();
const obj = {};
const foo = {};

ws.add(window);
ws.add(obj);
ws.has(window); // true
ws.has(foo); // false
ws.delete(window); // true
ws.has(window); // false
```


WeakSet 没有`size`属性，没有办法遍历它的成员

```纯文本
const ws = new WeakSet();

ws.size; // undefined
ws.forEach; // undefined
```


> WeakSet 不能遍历，是因为成员都是弱引用，随时可能消失，遍历机制无法保证成员的存在，很可能刚遍历结束，成员就取不到了


Map

基本用法

JavaScript 的对象，本质是键值对的集合，但传统上只能用字符串做键。

ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但键的范围不限于字符串，各种类型的值都可以作为键

```纯文本
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content');
m.get(o); // "content"
m.has(o); // true
m.delete(o); // true
m.has(o); // false
```


作为构造函数，Map 可以接受一个键值对数组作为参数。

```纯文本
const map = new Map([
	['name', 'stoat'],
	['title', 'Author']
]);

map.size; // 2
map.has('name'); // true
map.get('name'); // "stoat"
map.has('title'); // true
map.get('title'); // "Author"
```


事实上，任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构都可以当作`Map`构造函数的参数

```纯文本
const set = new Set([
	['foo', 1],
	['bar', 2]
]);
const m1 = new Map(set);
const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);

m1.get('foo'); // 1
m3.get('baz'); // 3
```


如果对一个键多次赋值，后面的值将覆盖前面的值

```纯文本
const map = new Map();

map
	.set(1, 'aaa')
	.set(1, 'bbb')
	.get(1);
// "bbb"
```


同样的值的两个实例，在 Map 结构中被视为两个键

```纯文本
const map = new Map();
const k1 = ['a'];
const k2 = ['a'];

map
	.set(k1, 111)
	.set(k2, 222);
map.get(k1); // 111
map.get(k2); // 222
```


如果 Map 的键是一个简单类型的值，则只要两个值严格相等，Map 将其视为一个键

```纯文本
let map = new Map();

map.set(-0, 123);
map.get(+0); // 123

map.set(true, 1);
map.set('true', 2);
map.get(true); // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined); // 3

map.set(NaN, 123);
map.get(NaN); // 123
```


Map 实例的属性和方法

- Map.prototype.constructor：构造函数
- Map.prototype.size：返回 Map 结构的成员总数
- Map.prototype.set(key, value)：添加值，返回 Map 结构本身
- Map.prototype.get(key)：读取值
- Map.prototype.has(key)：返回一个布尔值，表示某个键是否在当前 Map 结构中
- Map.prototype.delete(key)：删除值，返回成功与否的布尔值
- Map.prototype.clear()：清除值

```纯文本
const map = new Map();

map
	.set('foo', true)
	.set(1, {a: 1})
	.set(true, 22);

m.size; // 3
m.get(true); // 22
m.has(1); // true

m.delete(1); // true
m.has(1); // false

m.clear();
m.size; // 0
```


遍历方法

- Map.prototype.keys()：键名遍历器
- Map.prototype.values()：键值遍历器
- Map.prototype.entries()：键值对遍历器
- Map.prototype.forEach()：回调函数遍历每个成员

```纯文本
const map = new Map([
	['F', 'no'],
	['T', 'yes']
]);

for (const k of map.keys()){
	console.log(k);
}
// F T

for (const v of map.values()){
	console.log(v);
}
// no yes

for (const i of map.entries()){
	console.og(i);
}
// ["f", "no"] ["T", 'yes']
```


Map 结构的默认值遍历器接口是`entries`方法

```纯文本
const map = new Map([
	['F', 'no'],
	['T', 'yes']
]);

for (const i of map){
	console.log(i);
}
// ["F", "no"] ["T", "yes"]
```


Map 结构转为数组结构，比较快捷的方式是使用扩展运算符（`...`）

```纯文本
const map = new Map([
	[1, 'one'],
	[2, 'two'],
	[3, 'three']
]);

[...map.keys()]; // [1, 2, 3]
[...map.values()]; // ['one', 'two', 'three']
[...map.entries()]; // [[1, 'one'], [2, 'two'], [3, 'three']]
[...map]; // [[1, 'one'], [2, 'two'], [3, 'three']]
```


结合数组的`map`方法，`filter`方法，可以实现 Map 的遍历和过滤

```纯文本
const map = new Map();

map
	.set(1, 'a')
	.set(2, 'b')
	.set(3, 'c');
new Map([...map].filter(([k, v]) => k < 3));
// Map(2) {1 => "a", 2 => "b"}
new Map([...map].map(([k, v]) => [k * 2, `_${v}`]));
// Map(3) {2 => "_a", 4 => "_b", 6 => "_c"}
```


`forEach`方法，与数组的`forEach`方法类似，也可以实现遍历。可以接受第二个参数，用来绑定`this`

```纯文本
map.forEach((v, k, m) => console.log(v, k, m));
```


WeakMap

基本用法

WeakMap 结构与 Map 结构类似，也是用于生成键值对的集合。与 Map 结构的区别有两点

1. WeakMap 只接受对象作为键名
2. WeakMap 键名所指的对象都是弱引用（不计入垃圾回收机制）

```纯文本
const wm = new WeakMap();
const key = {foo: 1};

wm.set(key, 2);
wm.get(key); // 2

wm.set(2, 0); // TypeError: Invalid value used as weak map key
```


WeakMap 的设计目的在于，有时我们需要在某个对象上存放一些数据，但这会形成对这个对象的引用。一旦不需要这个对象，就必须手动删除这个引用，否则垃圾回收机制就不会释放

```纯文本
const e1 = document.getElementById('foo');
const e2 = document.getElementById('bar');
const arr = [
	[e1, 'foo'],
	[e2, 'bar']
];

// 不再需要 e1 和 e2 的时候
arr[0] = null;
arr[1] = null;
```


基本上，你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap

WeakMap 实例的属性和方法

- WeakMap.prototype.constructor：构造函数
- WeakMap.prototype.set(key, value)：添加值，返回 WeakMap 结构本身
- WeakMap.prototype.get(value)：获取值
- WeakMap.prototype.delete(value)：删除值，返回成功与否的布尔值
- WeakMap.prototype.has(value)：返回一个布尔值，表示该值是否为`WeakMap`的成员

```纯文本
	const wm = new WeakMap();
	const obj = {};

	wm.set(obj, 22);
	wm.get(obj); // 22
	wm.delete(obj); // true
	wm.has(obj); // false
```


WeakMap 没有遍历操作，即没有`keys()`、`values()`和`entries()`，也没有`size`属性。

