# class

class 基本语法

简介

```纯文本
class A {
  a = 1; // 实例属性
  b = () => this.a; // 实例方法
  static c = 2; // 静态属性
  static d = () => this.a; // 静态方法
  get e () { // 原型属性
      return this.a
  }
  f() { // 原型方法
      return this.#h();
  }
  #g = 3; // 私有属性
  #h = () => this.#g // 私有方法
}
```


类的由来

JavaScript 语言中，生成实例对象的传统方式是通过构造函数

```纯文本
function Point(x, y) {
	this.x = x;
	this.y = y;
}

Point.prototype.toString = function () {
	return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);
```


上面这种写法跟传统的面向对象语言差异很大



ES6 提供了更接近传统语言的写法，引进了类的概念，作为对象的模版



基本上，ES6 的

`class`

可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到

```纯文本
class Point {
	constructor(x, y) {
		this.x = x;
		this.y = y;
	}

	toString() {
		return `(${this.x}, ${this.y})`;
	}
}
```


ES6 的类完全可以看作构造函数的另一种写法

```纯文本
class Point {}

typeof Point; // 'function'
Point === Point.prototype.constructor; // true
```


使用的时候，也是直接对类使用`new`命令

```纯文本
class Bar {
	doStuff() {
		console.log('stuff');
	}
}
const b = new Bar();

b.doStuff(); // 'stuff'
```


构造函数的

`prototype`

属性，在 ES6 的类上面继续存在



事实上，类所有的方法都定义在类的

`prototype`

属性上

```纯文本
class Point {
	constructor() {}

	toString() {}

	toValue() {}
}
// 等同于
Point.prototype = {
	constructor() {},
	toString() {},
	toValue() {}
};
// 等同于
Object.assign(Point.prototype, {
	toString() {},
	toValue() {}
});
```


`prototype`对象的`constructor`属性，直接指向“类”的本身

```纯文本
Point.prototype.constructor === Point; // true
```


类的内部所有定义的方法，都不可枚举

```纯文本
Object.keys(Point.prototype); // []
Object.getOwnPropertyNames(Point.prototype); // ['constructor', 'toString']
```


constructor

`constructor`

方法是类的默认方法，通过

`new`

命令生成对象实例时，自动调用该方法



一个类必须有

`constructor`

方法，如果没有显式定义，一个空的

`constructor`

方法会被默认添加

```纯文本
class Point {}
// 等同于
class Point {
	constructor () {}
}
```


`constructor`方法默认返回实例对象（即`this`），完全可以指定返回另一个对象

```纯文本
class Foo {
	constructor() {
		return Object.create(null);
	}
}

new Foo() instanceof Foo; // false
```


类的实例

使用`new`命令生成类的实例

```纯文本
class Point {}
const point = new Point(2, 3);
```


实例的属性除非显式定义在其本身（即定义在`this`对象上），否则都是定义在原型上（即`class` 上）

```纯文本
class Point {
	constructor(x, y) {
		this.x = x;
		this.y = y;
	}

	toString() {
		return `(${this.x}, ${this.y})`;
	}
}
const point = new Point(2, 3);

point.toString(); // (2, 3)
point.hasOwnProperty('x'); // true
point.hasOwnProperty('y'); // true
point.hasOwnProperty('toString'); // false
point.__proto__.hasOwnProperty('toString'); // true
```


所有的实例共享一个原型对象

```纯文本
const p1 = new Point(2, 3);
const p2 = new Point(3, 2);

p1.__proto__ === p2.__proto__; // true
```


取值函数（getter）和存值函数（setter）

在“类”的内部可以使用`get`和`set`关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为

```纯文本
class MyClass {
	constructor() {}

	get prop() {
		return 'getter';
	}

	set prop(vlaue) {
		console.log(`setter: ${value}`);
	}
}
const inst = new MyClass();

inst.prop = 123; // setter: 123
inst.prop; // 'getter'
```


存值函数和取值函数是设置在属性的 Descriptor 对象上的

```纯文本
class CustomHTMLElement {
	constructor(element) {
		this.element = element;
	}

	get html() {
		return this.element.innerHTML;
	}

	set html(vlaue) {
		this.element.innerHTML = value;
	}
}
const descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, 'html');

'get' in descriptor; // true
'set' in descriptor; // true
```


属性表达式

类的属性名，可以采用表达式

```纯文本
const methodName = 'getArea';
class Square {
	constructor(length) {}
	[methodName]() {}
}
```


class 表达式

类也可以使用表达式定义

```纯文本
const MyClass = class Me {
	constructor() {
		return Me.name;
	}
};
const inst = new MyClass();

inst.getClassName(); // Me
```


可以简写成下面形式

```纯文本
const MyClass = class {};
```


注意点

- 严格模式

类和模块的内部，默认就是严格模式

- 不存在提升

类不存在变量提升

- name 属性

`name`属性总是返回紧跟在`class`关键字后面的类名

- Generator 方法

如果某个方法前面加上`*`，就表示该方法是一个 Generator 函数

- this 指向

类的方法内部如果含有`this`，它默认指向类的实例

静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承



如果在一个方法前，加上

`static`

关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就被称为“静态方法”

```纯文本
class Foo {
	static classMethod() {
		return 'hello';
	}
}

Foo.classMethod(); // 'hello'

const foo = new Foo();

fo.classMethod(); // TypeError: foo.classMethod is not a function
```


如果静态方法包含`this`关键字，这个`this`指的是类，而不是实例

```纯文本
class Foo {
	static bar() {
		this.baz();
	}
	static baz() {
		console.log('hello');
	}
	baz() {
		console.log('world');
	}
}
Foo.bar(); // hello
```


父类的静态方法可以被子类继承

```纯文本
class Foo {
	static classMethod() {
		return 'hello';
	}
}
class Bar extends Foo {}

Bar.classMethod(); // 'hello'
```


静态方法也可以从`super`对象上调用

```纯文本
class Foo {
	static classMethod() {
		return 'hello';
	}
}
class Bar extends Foo {
	static classMethod() {
		return super.classMethod() + ', too';
	}
}

Bar.classMethod(); // 'hello, too'
```


实例属性的新写法

实例属性除了定义在`constructor()`里面的`this`上面，可以定在类的最顶层

```纯文本
class IncreasingCounter {
	_count = 0;

	get value() {
		console.log('Getting the current value');
		return this._count;
	}

	increment() {
		this._count++;
	}
}
```


静态属性

静态属性指的是 class 本身的属性，即`Class.propName`

```纯文本
class Foo {
	static prop = 1;
}

Foo.prop; // 1
```


私有属性和方法

私有方法和私有属性，是只能在类的内部访问的方法和属性



在属性名之前添加

`#`

```纯文本
class IncreasingCounter {
	#count = 0;

	get value() {
		console.log('Getting the current value');
		return this.#count;
	}

	increment() {
		this.#count++;
	}
}
```


这种写法不仅可以写私有属性，还可以用来写私有方法

```纯文本
class Foo {
	#a;
	#b;
	constructor(a, b) {
		this.#a = a;
		this.#b = b;
	}
	#sum() {
		return #a + #b;
	}
	printSum() {
		console.log(this.#sum());
	}
}
```


私有属性也可以设置`getter`和`setter`方法

```纯文本
class Counter {
	#xValue = 0;

	constructor() {
		super();
	}

	get #x() {
		return #xValue;
	}

	set #x(value) {
		this.#xValue = value;
	}
}
```


new.target

`new`

是从构造函数生成实例对象的命令



ES6 引入了

`new.target`

属性，该属性一般用在构造函数当中



返回

`new`

命令作用于的那个构造函数



如果构造函数不是通过

`new`

命令或

`Reflect.construct`

调用，

`new.target`

会返回

`undefined`

```纯文本
function Person(name) {
	if (new.target !== undefined) {
		this.name = name;
	} else {
		throw new Error('必须通过 new 命令生成实例')
	}
}
```


Class 内部调用`new.target`，返回当前 Class

```纯文本
class Rectangle {
	constructor(length, width) {
		console.log(new.target === Rectangle);
		this.length = length;
		this.width = width;
	}
}
const obj = new Rectangle(3, 4); // true
```


子类继承父类时，`new.target`返回子类

```纯文本
class Rectangle {
	constructor(length, width) {
		console.log(new.target === Rectangle);
	}
}
class Square extends Rectangle {
	constructor(length, width) {
		super(length, width);
	}
}
const obj = new Square(3); // false
```


利用这个特点，可以写出不能独立使用，必须继承后才能使用的类

```纯文本
class Shape {
	constructor() {
		if (new.target === Shape){
			throw new Error('本类不能实例化');
		}
	}
}
class Rectangle extends Shape {
	constructor(length, width) {
		super();
	}
}
const x = new Shape(); // Error
const y = new Rectangle(3, 4);
```


