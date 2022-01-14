# proxy

Proxy

基本用法

Proxy 用于修改某些操作的默认行为，等同于在语言层面作出修改，所以属于一种“元编程”，即对编程语言进行编程

Proxy 可以理解为，在目标对象之前架设一层“拦截”，外界对该对象的访问，必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写

```纯文本
const obj = new Proxy({}, {
	get: function (target, propKey, receiver) {
		console.log(`getting ${propKey}`);
		return Reflect.get(target, propKey, receiver);
	},
	set: function (target, propKey, value, receiver) {
		console.log(`setting ${propKey}`);
		return Reflect.set(target, propKey, value, receiver);
	}
});

obj.count = 1;
// setting count
++obj
// getting count
// getting count
// 2
```


ES6 原生提供 Proxy 构造函数	，用来生成 Proxy 实例

```纯文本
const proxy = new Proxy(target, handler);
```


- target：所要拦截的目标对象
- handler：一个对象，用来定义拦截行为

```纯文本
const proxy = new Proxy({}, {
	get: function (target, propKey) {
		return 35;
	}
});

proxy.time; // 35
proxy.name; // 35
proxy.title; // 35
```


> 注意，要使得 Proxy 起作用，必须针对`Proxy`实例进行操作，而不是针对目标对象进行操作


如果`handler`没有设置任何拦截，就等于直接操作目标对象

```纯文本
const obj = {};
const proxy = new Proxy(obj, {});

proxy.a = 'b';
obj.a; // "b"
```


Proxy 实例也可以作为其他对象的原型对象

```纯文本
const proxy = new Proxy({}, {
	get: function (target, propKey) {
		return 35;
	}
});
const obj = Object.create(proxy);

obj.time; // 35
```


同一个拦截器函数，可以设置拦截多个操作

```纯文本
const handler = {
	get: function(target, name) {
		if (name === 'prototype'){
			return Object.prototype;
		}
		return `Hello, ${name}`;
	},
	apply: function (target, thisBinding, args) {
		return args[0];
	},
	constructor: function (target, args) {
		return {value: args[1]};
	}
};
const proxy = new Proxy(function (x, y){
	return args[0];
}, handler);

proxy(1, 2); // 1
new proxy(1, 2); // {value: 2}
proxy.prototype === Object.prototype; // true
proxy.foo === 'Hello, foo'; // true
```


Proxy 支持的拦截操作

1. get(target, propKey, receiver)：拦截对象属性的读取，如`proxy.foo`
2. set(target, propKey, value, receiver)：拦截对象属性的设置，如`proxy.foo = v`
3. has(target, propKey)：拦截`propKey in proxy`的操作，返回一个布尔值
4. deleteProperty(target, propKey)：拦截`delete proxy[propKey]`的操作，返回一个布尔值
5. ownKeys(target)：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组
6. getOwnPropertyDescriptor(target, propKey)：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象
7. defineProperty(target, propKey, propDesc)：拦截`Object.defineProperty(proxy, propKey, propDesc)`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值
8. preventExtensions(target)：拦截`Object.preventExtensions(proxy)`，返回一个布尔值
9. getPrototypeOf(target)：拦截`Object.getPrototypeOf(proxy)`，返回一个对象
10. isExtensible(target)：拦截`Object.isExtensible(proxy)`，返回一个布尔值
11. setPrototypeOf(target, proto)：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值
12. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`
13. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，如`new proxy(..args)`

拦截操作方法

get()

`get`方法用于拦截某个属性的读取操作。可以接受三个参数，依次为目标对象、属性名和 Proxy 实例本身（可选）

```纯文本
const person = {
	name: 'stoat'
};
const proxy = new Proxy(person, {
	get: function (target, propKey) {
		if (propKey in target){
			return target[propKey];
		} else {
			throw new ReferenceError(`Prop name "${propKey}" does not exist.`);
		}
	}
});

proxy.name; // "stoat"
proxy.age; // ReferenceError: prop "age" does not exist
```


set()

`set`方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和 Proxy 实例本身（可选）

```纯文本
const validator = {
	set: function (obj, prop, value) {
		if (prop === 'age'){
			if (!Number.isInteger(value)){
				throw new TypeError('The age is not an integer');
			}
			if (value > 200){
				throw new RangeError('The age seems invalid');
			}
		}
		obj[prop] = value;
	}
};
const person = new Proxy({}, validator);

person.age = 100;
person.age; // 100;
person.age = 'yong'; // error
person.age = 300; // error
```


apply()

`apply`方法拦截函数的调用、`call`和`apply`操作。可以接受三个参数，依次为目标对象、目标对象的上下文对象（this）和目标对象的参数数组

```纯文本
const target = function () {
	return 'I am the target';
};
const handler = {
	apply: function () {
		return 'I am the proxy';
	}
};
const proxy = new Proxy(target, handler);

p(); // "I am the proxy"
```


has()

`has`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。可以接受两个参数，依次是目标对象、需查询的属性名

```纯文本
const handler = {
	has(target, key) {
		if (key[0] === '_'){
			return false
		}
		return key in target;
	}
};
const target = {
	_prop: 'foo',
	prop: 'foo'
};
const proxy = new Proxy(target, handler);

'_prop' in proxy; // false
```


construct()

`construct`用于拦截`new`命令。可以接受三个参数，依次为目标对象，构造函数的参数对象和`new`作用的构造函数

```纯文本
const proxy = new Proxy(function (){}, {
	construct: function (target, args) {
		return {value: args[0] * 10};
	}
});
const p = new proxy(1);

p.value; // 10
```


deleteProperty()

`deleteProperty`方法用于拦截`delete`操作，如果这个方法抛出错误或者返回`false`，当前属性就无法被`delete`命令删除

```纯文本
const handler = {
	deleteProperty(target, key) {
		if (key[0] === '_'){
			throw new Error(`Invalid attempt to delete private "${key}" property`);
		}
		delete target[key];
		return false
	}
};
const target = {_prop: 'foo'};
const proxy = new Proxy(target, handler);

delete proxy._prop; // Error: Invalid attempt to delete private "_prop" property
```


defineProperty()

`defineProperty`方法拦截`Object.defineProperty操作`。可以接受三个参数，依次为目标对象、属性名和属性描述对象

```纯文本
const handler = {
	defineProperty(target, key, descriptor) {
		return false;
	}
};
const proxy = new Proxy({}, handler);

proxy.foo = 'bar';
proxy.foo; // undefined
```


getOwnPropertyDescriptor()

`getOwnPropertyDescriptor`方法拦截`Object.getOwnPropertyDescriptor`操作，返回属性描述对象

```纯文本
const = handler = {
	getOwnPropertyDescriptor(target, key) {
		if (key[0] === '_'){
			return;
		}
		return Object.getOwnPropertyDescriptor(target, key);
	}
};
const target = {_foo: 'bar', baz: 'tar'};
const proxy = new Proxy(target, handler);

Object.getOwnPropertyDescriptor(proxy, 'wat'); // undefined
Object.getOwnPropertyDescirptor(proxy, '_foo'); // undefined
Object.getOwnPropertyDescriptor(proxy, 'foo');
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```


getPrototypeOf()

`getPrototypeOf`方法主要用来拦截获取对象原型。具体来说，拦截下面这些操作

- `Object.prototype.__proto__`
- `Object.prototype.isPrototypeOf()`
- `Object.getPrototypeOf()`
- `Reflect.getPrototypeOf()`
- `instanceof`

```纯文本
const proto = {};
const proxy = new Proxy({}, {
	getPrototypeOf(target) {
		return proto;
	}
});

Object.getPrototypeOf(proxy) === proto; // true
```


isExtensible()

`isExtensible`方法拦截`Object.isExtensible`操作，返回布尔值

```纯文本
const proxy = new Proxy({}, {
	isExtensible(target) {
		return true;
	}
});

Object.isExtensible(proxy); // true
```


ownKeys()

`ownKeys`方法用来拦截对象自身属性的读取操作。具体来说，拦截一下操作

- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`
- `for...in`循环

```纯文本
const target = {
	a: 1,
	b: 2,
	c: 3
};
const handler = {
	ownKeys(target) {
		return ['a'];
	}
};
const proxy = new Proxy(target, handler);

Object.keys(proxy); // ['a']
```


preventExtensions()

`preventExtensions`方法拦截`Object.preventExtensions`操作，让一个对象变得不可扩展。返回一个布尔值

```纯文本
const proxy = new Proxy({}, {
	preventExtensions(target) {
		return true;
	}
});

Object.preventExtensions(proxy); // error
```


setPrototypeOf()

`setPrototypeOf`方法用来拦截`Object.setPrototypeOf`操作。

```纯文本
const handler = {
	setPrototypeOf(target, proto) {
		throw new Error('Changing the prototype is forbidden');
	}
};
const proto = {};
const target = function () {};
const proxy = new Proxy(target, handler);

Object.setPrototypeOf(proxy, proto); // error
```


Proxy.revocable()

返回一个可取消的 Proxy 实例

```纯文本
const target = {};
const handler = {};
const {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo; // 123
revoke.foo; // TypeError: Revoked
```


this 问题

虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象行为一致。主要原因就是在 Proxy 代理的情况下，目标对象内部的`this`关键字会指向 Proxy 代理

```纯文本
const target = {
	m() {
		console.log(this === proxy);
	}
};
const proxy = new Proxy(target, handler);

target.m(); // false
proxy.m(); // true
```


