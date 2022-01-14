# reflect

Reflect

基本用法

`Reflect`对象与`Proxy`对象一样，也是 ES6 位了操作对象而提供的新的 API

设计目的：

1. 将`Object`对象的一些明显属于语言内部的方法，放到`Reflect`对象上
2. 修改某些`Object`方法返回值，让其变得更合理
3. 让`Object`操作都变成函数行为
4. `Reflect`对象的方法与`Proxy`对象的方法一一对应

```纯文本
// ES5
Object.defineProperty(target, property, attributes);
Function.prototype.apply.call(Math.floor, undefined, [1.75]); // 1
// ES6
Reflect.defineProperty(target, property, attributes);
Reflect.apply(Math.floor, undefined, [1.75]); // 1

// ES5
'assign' in Object;
// ES6
Reflect.has(Object, 'assign');

Proxy(target, {
	set(target, name, value, receiver) {
		const success = Reflect.set(target, name, value, receiver);

		if (success){
			console.log('success');
		}
		return success;
	}
});
```


静态方法

`Reflect`对象一共有 13 个静态方法

- Reflect.apply(target, thisArg, args)
- Reflect.construct(target, args)
- Reflect.get(target, name, receiver)
- Reflect.set(target, name, value, receiver)
- Reflect.defineProperty(target, name, desc)
- Reflect.deleteProperty(target, name)
- Reflect.has(target, name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)

Reflect.get(target, name, receiver)

`Reflect.get`方法查找并返回`target`对象的`name`属性

```纯文本
const obj = {
	foo: 1,
	bar: 2,
	get baz() {
		return this.foo + this.bar;
	}
};

Reflect.get(obj, 'foo'); // 1
Reflect.get(obj, 'bar'); // 2
Reflect.get(obj, 'baz'); // 3
```


Reflect.set(target, name, value, receiver)

`Reflect.set`方法设置`target`对象的`name`属性等于`value`

```纯文本
const obj = {
	foo: 1,
	set bar(value) {
		return this.foo = value;
	}
};

obj.foo; // 1
Reflect.set(obj, 'foo', 2);
obj.foo; // 2
Reflect.set(obj, 'bar', 3);
obj.foo; // 3
```


Refelct.has(obj, name)

`Reflect.has`方法对应`name in obj`里面的`in`运算符

```纯文本
const obj = {foo: 1};

// ES5
'foo' in obj; // true
// ES6
Reflect.has(obj, 'foo'); // true
```


Reflect.deleteProperty(obj, name)

`Reflect.deleteProperty`方法等同于`delete obj[name]`，用于删除对象的属性

```纯文本
const obj = {foo: 'bar'};

// ES5
delete obj.foo;
// ES6
Reflect.deleteProperty(obj, 'foo');
```


Reflect.construct(target, args)

`Reflect.construct`方法等同于`new target(...args)`，这提供了一种不使用`new`，来调用构造函数的方法

```纯文本
function Greeting(name) {
	this.name = name;
}
// ES5
const instance = new Greeting('stoat');
// ES6
const instance = Reflect.construct(Greeting, ['stoat']);
```


Reflect.getPrototypeOf(obj)

`Reflect.getPrototypeOf`方法用于读取对象的`__proto__`，对应`Object.getPrototypeOf(obj)`

```纯文本
const obj = new FancyThing();

// ES5
Object.getPrototypeOf(obj) === FancyThing.prototype;
// ES6
Reflect.getPrototypeOf(obj) === FancyThing.prototype;
```


Reflect.setPrototypeOf(obj, newProto)

`Reflect.setPrototypeOf`方法用于设置目标对象的原型，对应`Object.setPrototypeOf(obj, newProto)`方法

```纯文本
const obj = {};

// ES5
Object.setPrototypeOf(obj, Array.prototype);
// ES6
Reflect.setPrototypeOf(obj, Array.prototype);
```


Reflect.apply(func, thisArg, args)

`Reflect.apply`方法等同于`Function.prototype.apply.call(func, thisArg, args)`，用于绑定`this`对象后执行函数

```纯文本
const ages = [11, 33, 12, 54, 18, 96];

// ES5
const y = Math.min.apply(Math, ages);
const o = Math.max.apply(Math, ages);
const t = Object.prototype.toString.call(y);

// ES6
const y = Reflect.apply(Math.min, Math, ages);
const o = Reflect.apply(Math.max, Math, ages);
const t = Reflect.apply(Object.prototype.toString, y, []);
```


Reflect.defineProperty(target, propertyKey, attributes)

`Reflect.defineProperty`方法基本等同于`Object.defineProperty`，用来给对象定义属性

```纯文本
function MyDate() {}

// ES5
Object.defineProperty(MyDate, 'now', {value: () => Date.now()});
// ES6
Reflect.defineProperty(MyDate, 'now', {value: Date.now()});
```


Reflect.getOwnPropertyDescriptor(target, propKey)

`Reflect.getOwnPropertyDescriptor`基本上等同于`Object.getOwnPropertyDescriptor`，用于得到对象属性的描述

```纯文本
const obj = {};

Object.defineProperty(obj, 'hidden', {
	value: true,
	enumerable: false	
});
// ES5
Object.getOwnPropertyDescriptor(obj, 'hidden');
// ES6
Reflect.getOwnPropertyDescriptor(obj, 'hidden');
```


Reflect.isExtensible(target)

`Reflect.isExtensible`方法对应`Object.isExtensible`，返回一个布尔值，表示对象是否可扩展

```纯文本
const obj = {};

// ES5
Object.isExtensible(obj); // true
// ES6
Reflect.isExtensible(obj); // true
```


Reflect.preventExtensions(target)

`Reflect.preventExtensions`对应`Object.preventExtensions`，用于让一个对象变为不可扩展

```纯文本
const obj = {};

// ES5
Object.preventExtensions(obj); // Object {}
// ES6
Reflect.preventExtensions(obj); // true
```


Reflect.ownKeys(target)

`Reflect.ownKeys`方法用于返回对象的所有属性，基本等于`Object.getOwnPropertyNames`与`Object.getOwnPropertySymbols`之和

```纯文本
const obj = {
	foo: 1,
	bar: 2,
	[Symbol.for('baz')]: 3,
	[Symbol.for('bing')]: 4
};

// ES5
Object.getOwnPropertyNames(obj); // ['foo', 'bar']
Object.getOwnPropertySymbols(obj); // [Symbol(baz), Symbol(bing)]
// ES6
Reflect.ownKeys(obj); // ["foo", "bar", Symbol(baz), Symbol(bing)]
```


