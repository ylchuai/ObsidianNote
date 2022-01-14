# object

对象的扩展

属性的简洁表示法

ES6 允许在大括号里面，直接写入变量和函数，作为对象的属性和方法

```纯文本
const foo = 'bar';
const baz = {foo};
baz; // {foo: "bar"}

function f(x, y) {
  return {x, y};
}
f(1, 2); // {x: 1, y: 2}
```


除了属性简写，方法也可以简写

```纯文本
const o = {
  method() {
    return 'Hello'
  }
}
```


下面是一个实际的例子

```纯文本
let birth = '2000/01/01';
const Person = {
  name: 'stoat',
  birth,
  hello() {
    console.log(`my name is ${this.name}`)
  }
}
```


适用情况

- 函数的返回值<br />function getPoint() {<br />const x = 1;<br />const y = 10;<br />return {x, y};

}

getPoint(); // {x: 1, y: 10}

- CommonJS 模块输出一组变量<br />let ms = {};<br />function getItem() {}<br />function setItem() {}<br />module.exports = {ms, getItem, setItem}
- 赋值器（settr）和取值器（getter）<br />const cart = {<br />_wheels: 4,<br />get wheels() {<br />return this._wheels;<br />},<br />set wheels() {<br />if (value < this.wheels) {<br />throw new Error('数值太小！');<br />}<br />this._wheels = value;<br />}<br />}

属性名不能用做构造函数

```纯文本
const obj = {
  f() {
    this.foo = 'bar';
  }
}
new obj.f(); // Error
```


属性名表达式

JavaScript 定义对象的属性，有两种方法

```纯文本
obj.foo = true;
obj['a' + 'bc'] = 23;
```


ES6 允许字面量定义对象时，把表达式放在方括号内

```纯文本
let propKey = 'foo';
let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123,
  ['h' + 'ello']() {
    return 'hi';
  }
};
```


属性名表达式与简洁表示法，不能同时使用

```纯文本
// 报错
const foo = 'bar';
const bar = 'abc';
const baz = {[foo]};

// 正确
const foo = 'bar';
const baz = {[foo]: 'abc'};
```


属性名表达式如果是一个对象，默认情况下会转化为字符串

```纯文本
const keyA = {a: 1};
const keyB = {b: 2};
const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject; // {[object Object]: "valueB"}
```


方法的 name 属性

函数的`name`属性，返回函数名。对象方法也是函数，因此也有`name`属性

```纯文本
const person = {
  sayName() {
    console.log('hello');
  }
};

person.sayName.name; // 'sayName'
```


如果，对象的方法使用了取值函数和存值函数，则`name`属性不在该方法上面，而是该方法的属性的描述对象的`get`和`set`属性上面，返回值是方法名前加上`get`和`set`

```纯文本
const obj = {
  get foo() {},
  set foo(x) {}
};

obj.foo.name; // TypeError: Cannot read property 'name' of undefined

const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');

descriptor.get.name; // "get foo"
descriptor.set.name; // "set foo"
```


有两种特殊情况：`bind`方法创造的函数，`name`属性返回`bound`加上原函数的名字；`Function`构造函数创造的函数，`name`属性返回`anonymous`

```纯文本
(new Function()).name; // anonymous
const doSomething = () => {};
doSomething.bind().name; // "bound doSomething"
```


如果对象的方法是一个`Symbol`值，`name`属性返回这个`Symbol`值的描述

```纯文本
const key1 = Symbol('descroption');
const key2 = Symbol();
const obj = {
  [key1]() {},
  [key2]() {}
};

obj[key1].name; // "[description]"
obj[key2].name; // ""
```


属性的可枚举性和遍历

可枚举性

对象的每个属性都有一个描述对象，用来控制该属性的行为

```纯文本
let obj = {foo: 123};
Object.getOwnPropertyDescriptor(obj, 'foo');

// {
//  value: 123,
//  writable: true,
//  enumerable: true,
//  configurable: true
// }
```


描述对象的`enumerable`属性，称为“可枚举性”

目前，有四个操作会忽略`enumerable`为`false`的属性

1. `for...in`：只遍历对象自身的和继承的可枚举的属性
2. `Object.keys()`：返回对象自身的所有可枚举的属性的键名
3. `JSON.stringify()`：只串化对象自身的可枚举属性
4. `Object.assign()`：只拷贝对象自身的可枚举属性

实际上，引入“可枚举”这个概念的最初目的，就是让某些属性可以规避掉`for...in`操作，不然所有内部属性和方法都会被遍历到

```纯文本
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable; // false
Object.getOwnPropertyDescriptor([], 'length').enumerable; // false
```


ES6 规定，所有 Class 原型的方法都是不可枚举的

```纯文本
Object.getOwnPropertyDescriptor(class Foo{foo(){}}.prototype, 'foo').enumerable; // false
```


属性的遍历

1. `for...in`

循环遍历对象自身和继承的可枚举属性

1. `Object.keys()`

返回一个数组，包含对象自身的所有可枚举属性的键名

1. `Object.getOwnPropertyNames()`

返回一个数组，包含对象自身所有属性的键名

1. `Object.getOwnPropertySymbols()`

返回一个数组，包含对象自身所有的 Symbol 属性的键名

1. `Reflect.ownKeys()`

返回一个数组，包含对象自身所有的键名

以上 5 种方法遍历对象的键名，都遵守同样的属性遍历的测序规则

- 首先遍历所有数值键，按照数值升序排列
- 其次遍历所有字符串键，按照加入时间升序排列
- 最后遍历所有 Symbol 键，按照加入时间升序排列<br />Reflect.ownKeys({<br />[Symbol()]: 0,<br />b: 0,<br />10: 0,<br />2: 0,<br />a: 0<br />});<br />// ['2', '10', 'b', 'a', Symbol()]

super 关键字

我们知道，`this`关键字总是指向函数所在的当前对象。ES6 又新增了一个类似的关键字`super`，指向当前对象的原型对象

```纯文本
const proto = {foo: 'hello'};
const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find(); // "hello"
```


`super`关键字表示原型对象时，只能用在对象的方法中，用在其他地方都会报错

```纯文本
// 报错
const obj = {
  foo: super.foo
};

// 报错
const obj = {
  foo: () => super.foo
};

// 报错
const obj = {
  foo: function () {
    return super.foo;
  }
}
```


> 目前，只有对象方法的简写法可以让 JavaScript 引擎确认，定义的是对象的方法


JavaScript 引擎内部，`super.foo`等同于`Object.getPrototypeOf(this).foo`（属性）或`Object.getPrototypeOf(this).foo.call(this)`（方法）

```纯文本
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  }
};
const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
};

Object.setPrototypeOf(obj, proto);
obj.foo(); // "world"
```


对象的扩展运算符

解构赋值

对象的解构赋值用于从一个对象取值，相当于将目标对象自身的所有可遍历的、但尚未被读取的属性，分配到指定的对象上。所有的键和它们的值，都会拷贝到新对象上

```纯文本
let {x, y, ...z} = {x: 1, y: 2, a: 3, b: 4};

x; // 1
y: // 2
z: // {a: 3, b: 4}
```


解构赋值的拷贝是浅拷贝，即如果一个键的值是复合类型的值，那么解构赋值拷贝的是这个值的引用，而不是这个值的副本

```纯文本
let obj = {a: {b: 2}};
let {...x} = obj;

obj.a.b = 3;
x.a.b; // 3
```


扩展运算符的的解构赋值，不能复制继承自原型对象的属性

```纯文本
let o1 = {a: 1};
let o2 = {a: 2};

o2.__proto__ = o1;

let {...o3} = o2;

o3; // {b: 2}
o3.a; // undefined
```


扩展运算符

对象的扩展运算符（`...`）用于取出参数对象的所有可遍历属性，拷贝到当前对象中

```纯文本
let z = {a: 3, b: 4};
let n = {...z};

n; // {a: 3, b: 4}
```


由于数组是特殊的对象，所以对象的扩展运算符也可以用于数组

```纯文本
let foo = {...['a', 'b', 'c']};

foo; // {0: "a", 1: "b", 2: "c"}
```


如果扩展运算符后面不是对象，则会自动将其转化为对象

```纯文本
// 等同于 {...Object(1)}
{...1}; // {}

// 等同于 {...Object(true)}
{...true}; // {}
```


如果扩展运算符后面是字符串，它会自动转化为类数组对象

```纯文本
{...'hello'};
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```


对象的扩展运算符等同于使用`Object.assign()`方法

```纯文本
let aClone = {...a};
// 等同于
let aClone = Object.assign({}, a);
```


如果想完整克隆一个对象，还拷贝对象原型的属性，可以采用下面的方法

```纯文本
// 写法一
const clone1 = {
  __proto__: Object.getPrototypeOf(obj),
  ...obj
};

// 写法二
const clone2 = Object.assign(
  Object.create(Object.getPrototypeOf(obj)),
  obj
);

// 写法三
const clone3 = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```


扩展运算符还可以用来合并对象

```纯文本
{...a, ...b}
```


如果用户自定义的属性，放在扩展运算符后面，则扩展运算符内部的同名属性会被覆盖掉

```纯文本
{...{a: 1, b: 2}, a: 0, c: 0};
// {a: 0, b: 2, c: 0}
```


如果把自定义属性放在扩展运算符前面，就变成了设置新对象的默认属性

```纯文本
{a: 0, c: 0, ...{a: 1, b:1}};
// {a: 1, c: 0, b: 1}
```


对象的扩展运算符后面可以跟表达式

```纯文本
const obj = {
  ...(x > 1 ? {a: 1} : {}),
  b: 2
};
```


扩展运算符的参数对象之中，如果有取值函数`get`，这个函数是会执行的

```纯文本
{...{get x() {return 2}}};
// {x: 2}
```


链判断运算符

编程实务中，如果要读取对象内部的某个属性，往往需要先判断一下该对象存不存在。

```纯文本
const name = (mes && mes.body && mes.body.user && mes.body.user.name) || 'default';
```


ES2020 引入了“链判断运算符”`?.`

```纯文本
const name = mes?.body?.user?.name || 'default'
```


判断对象方法是否存在，如果存在就立即执行

```纯文本
const obj = {say() {return 1}};

obj.say?.(); // 1
obj.do?.(); // undefined
```


对于那些没有实现的方法，这个运算符尤其重要

```纯文本
if (myForm.checkValidity?.() === false){
  // 表单校验失败
  return;
}
```


下面是一些常见的使用形式

```纯文本
a?.b;
// 等同于
a == null ? undefined : a.b;

a?.[x]
// 等同于
a == null ? undefined : a[x];

a?.b()
// 等同于
a == null ? undefined : a.b();

a?.()
// 等同于
a == null ? undefined : a()
```


注意点：

1. 短路机制<br />a?.[++x];<br />// 等同于<br />a == null ? undefined : a[++x];

如果`a`是`undefined`或者`null`，那么`x`不会进行递增运算

1. delete 运算符<br />delete a?.b;<br />// 等同于<br />a == null ? undefined : delete a.b;

如果`a`是`undefined`或`null`，那么不会进行`delete`运算

1. 括号的影响<br />(a?.b).c;<br />// 等同于<br />(a == null ? undefined : a.b).c;

链判断运算符对圆括号外部没有影响

1. 报错场合<br />// 构造函数<br />new a?.();<br />new a?.b();<br />// 链判断运算符的右侧有模版字符串<br />a?.`{b}`;<br />a?.b`{c}`;<br />// 链判断运算符的左侧是 super<br />super?.();<br />super?.foo<br />// 链运算符用于赋值运算符左侧<br />a?.b = c
2. 右侧不得为十进制数字

Null 判断运算符

读取对象属性的时候，如果某个属性的值是`undefined`或者`null`，有时需要为它们设置默认值

```纯文本
response.settings.headerText || 'Hello, world';
```


ES2020 引入了一个 Null 判断运算符`??`。它的行为类似`||`，但是只有运算符左侧的值为`null`或`undefined`时，才会返回右侧的值

```纯文本
response.settings.headerText ?? 'Hello, world';
```


这个运算符的一个目的，就是和链判断运算符`?.`配合使用，为`null`或`undefined`的值设置默认值

```纯文本
response?.settings?.headerText ?? 'Hello, world';
```


这个运算符很适合判断函数参数是否赋值

```纯文本
function Component(props) {
  const enable = props.enabled ?? true;
  // ...
}
```


`??`有一个运算优先级问题，如果多个逻辑运算符一起使用，必需使用括号表面优先级

```纯文本
 // 报错
 0 && 1 ?? 1;

 //正确
 (0 && 1) ?? 1;
```


