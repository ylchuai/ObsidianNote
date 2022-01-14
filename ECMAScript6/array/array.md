# array

数组的扩展

扩展运算符

扩展运算符是`...`。它好比 rest 参数的逆运算，将一个数组转化为用逗号分隔参数序列

```纯文本
console.log(...[1,2,3]);
// 1 2 3
```


该运算符主要用于函数调用

```纯文本
function push(array, ...items) {
  array.push(...items)
}

function add(x, y) {
  return x + y;
}
const numbers = [4, 38];

add(...numbers); // 42
```


扩展运算符可以与正常函数参数结合使用，非常灵活

```纯文本
function f(v, w, x, y, z) {}
const args = [0, 1];

f(-1, ...args, 2, ...[3]);
```


扩展运算符后面还可以放表达式

```纯文本
const arr = [
  ...(x > 0 ? ['a'] : []),
  'b',
];
```


由于扩展运算符可以展开数组，所以不再需要`apply`方法，将数组的转化为函数的参数了

```纯文本
// ES5 的写法
function f(x, y, z) {}
var args = [0, 1, 2];

f.apply(null, args);

// ES6 的写法
function f(x, y, z) {}
let args = [0, 1, 2];

f(...args);
```


扩展运算的应用

1. 赋值数组<br />const a1 = [1, 2];<br />const a2 = [...a1];<br />const [...a2] = a1;
2. 合并数组<br />const arr1 = ['a', 'b', 'c'];<br />const arr2 = ['d', 'e'];<br />const arr3 = ['f'];<br />[...arr1, ...arr2, ...arr3];
3. 与解构赋值结合<br />const [first, ...rest] = [1, 2, 3];<br />first; // 1<br />rest; // [2, 3]
4. 扩展字符串<br />[...'hello'];<br />// ['h', 'e', 'l', 'l', 'o']
5. 实现了 Iterator 接口的对象<br />Number.prototype[Symbol.iterator] = function *() {<br />let i = 0;<br />let num = this.valueOf();<br />}<br />console.log([...5]); // [0, 1, 2, 3, 4]
	```纯文本
	 while(i < num) {
	   yield i++;
	 }
	```
	
6. Map、Set 和 Generator 函数<br />let map = new Map([<br />[1, 'one'],<br />[2, 'two'],<br />[3, 'three'],<br />]);<br />let arr = [...map.keys()]; // [1, 2, 3]<br />const go = function *() {<br />yield 1;<br />yield 2;<br />yield 3;<br />}<br />[...go()]; // [1, 2, 3]

> 任何定义了遍历器接口的对象，都可以使用扩展运算符


Array.from()

`Array.from`方法用于将两类对象转化为真正的数组：类数组对象和可遍历对象

```纯文本
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};

// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']
// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```


实际应用中，常见的类数组对象是 DOM 操作返回的 NodeList 集合，以及函数内部的 arguments 对象

```纯文本
// NodeList 对象
let ps = document.querySelectorAll('p');
Array.from(ps).filter(p => {
  return p.textContent.length > 100;  
});

// arguments 对象
function foo() {
  var args = Array.from(arguments);
}
```


只要部署了 Iterator 接口的数据结构，`Array.from`都能将其转化为数组

```纯文本
Array.from('hello');
// ['h', 'e', 'l', 'l', 'o']

let nameSet = new Set*(['a', 'b']);
Array.from(nameSet); // ['a', 'b'];
```


如果参数是一个真正的数组，`Array.from`会返回一个一摸一样的新数组

```纯文本
Array.from([1, 2, 3]);
// [1, 2, 3]
```


值得提醒的是，扩展运算符(`...`)也可以将类数组对象转化为数组

```纯文本
function foo() {
  const args = [...arguments];
}

[...document.querySelectorAll('p')];
```


所谓类数组对象，本质只有一点，即必须有`length`属性

```纯文本
Array.from({ length: 3 });
// [undefined, undefined, undefined]
```


对于没有部署该方法的浏览器，可以用`Array.prototype.slice`

```纯文本
const toArray = (() => Array.from ? Araay.from : obj => [].slice.call(obj))()
```


`Array.from`还可以接受第二个参数，作用类似于数组的`map`方法，用来对每个元素进行处理，将处理后的值放入返回的数组

```纯文本
Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], x => x * x);
// [1, 4, 9]
```


Array.of()

`Array.of()`方法用于将一组值，转化为数组

```纯文本
Array.of(3, 11, 8); // [3, 11, 8]
Array.of(3); // [3]
Array.of(3).length; // 1
```


`Array.of()`基本上可以替代`Array()`或`new Array()`，并且不存在由于参数不同而导致的重载

```纯文本
Array.of(); // []
Array.of(undefined); // [undefined]
Array.of(1); // [1]
Array.of(1, 2); // [1, 2]
```


`Array.of()`可以用下面的代码模拟实现

```纯文本
function ArrayOf() {
  return [].slice.call(arguments);
}
```


copyWithin()

数组实例的`copyWithin（）`方法，在当前数组内部，将制定位置的成员复制到其他位置（会覆盖原有成员），返回当前数组。也就是，使用这个方法，会修改当前数组

```纯文本
Array.prototype.copyWithin(target, start = 0, end = this.length)
```


- target(必需)：从该位置开始替换数据
- start(可选：从该位置开始读取数据
- end(可选)：到该位置停止读取数据<br />let arr = [1, 2, 3, 4, 5];<br />arr.copyWithin(0, 2, 4); // [3, 4, 3, 4, 5]<br />arr; // [3, 4, 3, 4, 5]

下面是更多例子

```纯文本
[1, 2, 3, 4, 5].copyWithin(0, 3, 4); // [4, 2, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(0, -2, -1); // [4, 2, 3, 4, 5]

[].copyWithin.call({ length: 5, 3: 1}, 0, 3); // {0: 1, 3: 1, length: 5}

let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2); // int32Array [3, 4, 5, 4, 5]

[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4); // Int32Array [4, 2, 3, 4, 5]
```


find() 和 findIndex()

数组实例的`find`方法，用于找出符合条件的第一个数组成员。如果没有符合条件的成员，则返回`undefined`

```纯文本
[1, 4, -5, 10].find(n => n < 0); // -5

[1, 5, 10].find((v, i, a) => v > 5); // 10
```


`find`方法的回调函数可以接受三个参数，依次为当前的值、当前的位置和原数组

数组实例的`findIndex`，返回第一个符合条件成员的位置。如果没有符合条件的成员，则返回`-1`

```纯文本
[1, 5, 10].findIndex((v, i, a) => v > 5); // 2
```


这两个方法都可以接受第二个参数，用来绑定函数的`this`对象

```纯文本
[1, 5, 10].find(function (v) {
  return v > this.age;  
}, {age: 5});
// 10
```


这两个方法都可以发现`NaN`，弥补了数组的`indexOf`方法的不足

```纯文本
[NaN].indexOf(NaN); // -1
[NaN].findIndex(v => Object.is(v, NaN)); // 0
```


fill()

数组实例的`fill`方法，使用给定值填充一个数组

```纯文本
['a', 'b', 'c'].fill(7); // [7, 7, 7]
new Array(3).fill(7); // [7, 7, 7]
```


`fill`方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置

```纯文本
['a', 'b', 'c'].fill(7, 1, 2); // ['a', 7, 'c']
```


如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象

```纯文本
let arr = new Array(3).fill({a: 0});

arr; // [{a: 0}, {a: 0}, {a: 0}]
arr[0].a = 1;
arr; // [{a: 1}, {a: 1}, {a: 1}]
```


entries()、keys()和values()

ES6 提供这三个方法用来遍历数组。它们都返沪一个遍历器对象，可以用`for...of`循环进行遍历。`keys`是对键名的遍历，`values`是对键值的遍历，`entries`是对键值的遍历

```纯文本
for (let index of ['a', 'b'].keys()){
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```


includes()

数组实例的`includes`方法，判断数组是否包含给定的值

```纯文本
[1, 2, 3].includes(3); // true
[1, 2, 3].includes(4); // false
[1, 2, NaN].includes(NaN); // true
```


该方法的第二个参数表示搜索的起始位置，默认为`0`

```纯文本
[1, 2, 3].includes(2, 1); // true
[1, 2, 3].includes(2, -2); // true
```


flat()、flatMap()

数组的成员有时还是数组，数组实例的`flat`方法用于将嵌套的数组"拉平"，变成一维数组。该方法返回一个新数组，对原数据没有影响

```纯文本
[1, 2, [3, 4]].flat(); // [1, 2, 3, 4]
```


`flat`方法默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以将`flat`方法的参数写成一个整数，表示想要“拉平”的层数，默认为`1`

```纯文本
[1, 2, [3, [4, 5]]].flat(); // [1, 2, 3, [4, 5]]
[1, 2, [3, [4, 5]]].flat(2); // [1, 2, 3, 4, 5]
```


如果不管多少层嵌套，都要转成一维数组，可以用`Infinity`关键字作为参数

```纯文本
[1, 2, [3, [4, 5]]].flat(Infinity); // [1, 2, 3, 4, 5]
```


如果原数组有空位置，`flat`方法会跳过空位置

```纯文本
[1, 2, , 4].flat(); // [1, 2, 4]
```


`flatMap`方法对原数组的每个成员执行一个函数，然后对返回值组成的数组执行`flat`方法。该方法返回一个新数组，不改变原数组

```纯文本
[2, 3, 4].flatMap(v => [v * 2, v ** 2]); // [4, 4, 6, 9, 8 ,16]
```


`flatMap`方法只能展开一层数组

```纯文本
[2, 3, 4].flatMap(v => [[v * 2]]); // [[4], [6], [8]]
```


`flatMap`方法的参数是一个遍历函数，该函数可以接受三个参数，分别是当前数组成员、当前数组成员的位置，原数组。`flatMap`方法还可以有第二个参数，用来绑定遍历函数里面的`this`

```纯文本
arr.flatMap(function callback(currentValue[, index[, array]]){}[, thisArg])
```


数组的空位

数组的空位指，数组的某一个位置没有任何值

```纯文本
Array(3); // [, , ,]
```


> 空位不是`undefined`，一个位置的值等于`undefined`，依然是有值的


ES5 对空位的处理

- `forEach()`, `filter()`, `reduce()`, `every()`和`some()`会跳过空位
- `map()`会跳过空位，但会保留这个值
- `join()`和`toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串

```纯文本
[, 'a'].forEach((x, i) => console.log(i)); // i
['a', , 'b'].filter(x => true); // ['a', 'b']
[, 'a'].every(x => x === 'a'); // true
[1, , 2].reduce((x, y) => x + y); // 3
[, 'a'].some(x => x !== 'a'); // false
[, 'a'].map(x => 1); // [, 1]
[, 'a', undefined, null].join('#'); // "#a##"
[, 'a', undefined, null].toString(); // ",a,,"
```


ES6 则明确将空位转为`undefind`

```纯文本
Array.from(['a', , 'b']); // ['a', undefined, 'b']
[...['a', , 'b']]; // ['a', undefined, 'b']
[, 'a', 'b', ,].copyWithin(2, 0); // [, 'a', , 'a']
new Array(3).fill('a'); // ['a', 'a', 'a']

let arr = [, ,];

for (let i of arr){
  console.log(1);
}
// 1
// 1
```


排序稳定性

排序稳定性是排序算法的重要属性，指的是排序关键字相同的项目，排序前后的顺序保持不变

