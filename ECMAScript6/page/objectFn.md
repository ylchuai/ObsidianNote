# objectFn

对象的新增方法

Object.is()

ES5 比较两个值是否相等，只有两个运算符：相等运算符（`==`）和严格相等运算符（`===`）。它们都有缺点，前者会自动转化数据类型，后者的`NaN`不等于自身，以及`+0`等于`-0`

ES6 使用`Object.is`方法比较两个值是否严格相等，与严格相等运算符（`===`）的行为基本一致

```纯文本
Object.is('foo', 'foo'); // true
Object.is({}, {}); // false
```


不同之处有两个：`+0`不等于`-0`；`NaN`等于自身

```纯文本
+0 === -0； // true
NaN === NaN; // false

Object.is(+0, -0); // false
Object.is(NaN, NaN); // true
```


ES5 可以通过下面的代码，部署`Object.is`

```纯文本
Object.defineProperty(Object, 'is', {
	value: function(x, y) {
		if (x === y){
			return x !== 0 || 1 / x === 1 / y;
		}
		return x !== x && y !== y;
	},
	configurable: true,
	enumerable: false,
	writable: true
});
```


Object.assign()

`Object.assign`方法用于对象的合并，将源对象的所有可枚举属性，复制到目标对象

```纯文本
const target = {a: 1, b: 1};
const source1 = {b: 2, c: 4};
const source2 = {c: 3};

Object.assign(target, source1, source2);
target; // {a: 1, b: 2, c: 3}
```


字符串的包装对象，会产生可枚举属性

```纯文本
Object.assign({}, 'hello', true, null);
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```


属性名为 Symbol 值的属性，也会被`Object.assign`拷贝

```纯文本
Object.assign({a: 1}, {[Symbol('c')]: 2});
// {a: 1, Symbol(c): 2}
```


> `Object.assign`拷贝的属性是有限制的，只拷贝对象自身可枚举属性


注意点

1. 浅拷贝

`Object.assign`方法实现的是浅拷贝。如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用

```纯文本
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b; // 2
```


1. 同名属性的替换

`Object.assign`方法遇到同名属性的处理方法事替换，而不是添加

```纯文本
const target = {a: {b: 'c', d: 'e'}};
const source = {a: {b: 'hello'}};

Object.assign(target, source); // {a: {b: 'hello'}}
```


1. 数组的处理

`Object.assign`可以用来处理数组，但会把数组视为对象

```纯文本
Object.assign([1, 2, 3], [4, 5]);
// [4, 5, 3]
```


1. 取值函数的处理

`Object.assign`只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制

```纯文本
const source = {
	get foo() {
		return 1;
	}
};
const target = {};

Object.assign(target, source); // {foo: 1}
```


常见用途

1. 为对象添加属性<br />class Point {<br />constructor(x, y) {<br />Object.assign(this, {x, y});<br />}<br />}
2. 为对象添加方法<br />Object.assign(SomeClass.prototype, {<br />someMethod(arg1, arg2) {},<br />anotherMethod() {}<br />});<br />// 等同于<br />SomeClass.prototype.someMethod = function (arg1, arg2) {};<br />someClass.prototype.anotherMethod = function () {};
3. 克隆对象<br />function clone(origin) {<br />return Object.assign({}, origin);<br />}
4. 合并多个对象<br />const merge = (target, ...sources) => Object.assign(target, ...sources);
5. 为属性指定默认值<br />const DEFAULTS = {<br />logLevel: 0,<br />outputFormat: 'html'<br />};<br />function processContent(options) {<br />options = Object.assign({}, DEFAULTS, options);<br />}
	```纯文本
	 console.log(options);
	```
	

Object.getOwnPropertyDescriptors()

ES5 的`Object.getOwnPropertyDescriptor`方法返回某个对象属性的描述对象。<br />ES2017 引入`Object.getOwnPropertyDescriptors`方法，返回指定对象所有自身属性的描述对象

```纯文本
Object.getOwnPropertyDescriptor({a: 1, find() {}});
// {
// 	a: {
// 		value: 1
// 		writable: true
// 		enumerable: true
// 		configurable: true
// 	},
// 	find: {
// 		value: ƒ find()
// 		writable: true
// 		enumerable: true
// 		configurable: true
// 	}
// }
```


该方法的实现非常容易

```纯文本
function getOwnPropertyDescriptors(obj) {
	const result = {};

	for (let key of Reflect.ownKeys(obj)){
		result[key] = Object.getOwnPropertyDescriptor(obj, key);
	}
	return result;
}
```


该方法的引入目的，主要是为了解决`Object.assign`方法无法正确拷贝`get`属性和`set`属性的问题

```纯文本
const source = {
	set foo(value) {
		console.log(value);
	}
};
const target1 = {};

Object.assign(target1, source);
Object.getOwnPropertyDescriptor(target1, 'foo');
//	{
//		value: undefined
//		writable: true
//		enumerable: true
//		configurable: true
//	}
```


`Object.assign`方法总是拷贝一个属性的值，而不会拷贝它背后的赋值方法或者取值方法

这时，`Object.getOwnPropertyDescriptors`方法配合`Object.defineProperties`方法，就可以实现正确拷贝

```纯文本
const source = {
	set foo(value) {
		console.log(value);
	}
};
const target2 = {};

Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo');
//	{
//		get: undefined
//		set: ƒ foo(value)
//		enumerable: true
//		configurable: true
//	}
```


`Object.getOwnPropertyDescriptors`方法的另一个用处，是配合`Object.create`方法，将对象属性克隆到一个新对象

```纯文本
const clone = Object.create(
	Object.getPrototypeOf(obj),
	Object.getOwnPropertyDescriptor(obj)
);
```


`Object.getOwnPropertyDescriptors`方法可以实现一个对象继承另一个对象

```纯文本
// __proto__
const obj = {
	__proto__: prot,
	foo: 123
};

// create
const obj = Object.create(prot);
obj.foo = 123;

const obj = Object.assign(Object.create(prot), {foo: 123});

// getOwnPropertyDescriptors
const obj = Object.create(
	prot,
	Object.getOwnPropertyDescriptors({foo: 123})
);
```


**proto**, Object.setPropertyOf(), Object.getPropertyOf()

1. `__proto__`属性，用来读取或设置当前对象的原型对象<br />const obj = {<br />method: function (){}<br />};<br />obj.**proto** = someOtherObj;

> 标准明确规定，只有浏览器必需部署这个属性，其他运行环境不一定要部署。因此，不推荐使用这个属性


1. `Object.setPrototypeOf`方法，用来设置一个对象的原型对象，返回参数本身<br />Object.setPrototypeOf(object, prototype);
2. `Object.getPrototypeOf`方法，用来读取一个对象的原型对象<br />Object.getPrototypeOf(object);

Object.keys(), Object.values(), Object.entries()

`Object.keys`方法返回一个数组，成员是参数对象自身的所有可遍历属性的键名

```纯文本
const obj = {foo: 'bar', baz: 42};

Object.keys(obj); // ["foo", "baz"]
```


`Object.values`方法返回一个数组，成员是参数对象自身的所有可遍历属性的键值

```纯文本
const obj = {foo: 'bar', baz: 42};

Object.values(obj); // ["bar", 42]
```


`Object.entries`方法返回一个数组，成员是参数对象自身所有可遍历属性的键值对数组

```纯文本
const obj = {foo: 'bar', baz: 42};

Object.entries(obj); //[["foo", "bar"], ["baz", 42]]
```


如果原对象的属性是一个 Symbol 值，该属性会被忽略

```纯文本
Object.entries({[Symbol()]: 123, foo: 'abc'});
// [["foo", "abc"]]
```


Object.fromEntries()

`Object.fromEntries`方法是`Object.entries`方法的逆操作，用于将一个键值对数组转化为对象

```纯文本
Object.fromEntries([
	['foo', 'bar'],
	['baz', 42]
]);
// {foo: "bar", baz: 42}
```


该方法的主要目的，是将键值对的数据结构还原为对象，因此特别适合将 Map 结构转化为对象

```纯文本
const entries = new Map([
	['foo', 'bar'],
	['baz', 42]
]);
Object.fromEntries(entries); // {foo: "bar", baz: 42}

const map = new Map().set('foo', true).set('bar', false);
Object.fromEntries(map); // {foo: true, bar: false}
```


该方法的一个用处是配合`URLSearchParams`对象，将查询字符串转化为对象

```纯文本
Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'));
// {foo: "bar", baz: "qux"}
```


