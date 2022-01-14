# classExtends

Class 的继承

简介

Class 可以通过`extends`关键字继承，这比 ES5 通过修改原型链实现继承，要清晰和方便很多

```纯文本
class Point {}
class ColorPoint extends Point {}
```


子类必须在

`constructor`

方法中调用

`super`

方法



子类自己的

`this`

对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法



然后再对其加工，加上实例自己的属性和方法

```纯文本
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y);
    this.color = color;
  }
}
```


上面代码中，`constructor`和`toString`方法中，都出现了`super`关键字，它在这里表示父类的构造函数，用来新建父类的`this`对象

ES5 的继承，实质上是先创建子类的实例对象

`this`

，然后再将父类的方法添加到

`this`

上（

`Parent.aplly(this)`

）



ES6 的继承机制完全不同，实际上是先将父类实例对象的属性和方法加到

`this`

上面，然后再用子类的构造函数修改

`this`



如果子类没有定义

`constructor`

方法，这个方法会被默认添加

```纯文本
class ColorPoint extends Point {}
// 等同于
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}
```


在子类构造函数中，只有调用

`super`

之后，才可以使用

`this`

关键字



因为子类实例的构建，基于父类实例，只有

`super`

方法才能调用父类实例

```纯文本
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}
class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // Error
    super(x, y);
    this.color = color;
  }
}
const cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint; // true
cp instanceof Point; // true
```


父类的静态方法也会被子类继承

```纯文本
class A {
  static hello = () => 'hello';
}
class B extends A {}

B.hello(); // 'hello'
```


Object.getPrototypeOf()

从子类获取父类

```纯文本
Object.getPrototypeOf(ColorPoint) === Point; // true
```


super

`super`关键字，既可以当作函数使用，也可以当作对象使用

作为函数调用时，代表父类的构造函数

```纯文本
class A {}
class B extends A {
  constructor() {
    super();
  }
}
```


`super`虽然代表了父类`A`的构造函数，但返回的是子类`B`的实例，即`super`内部的`this`指的是`B`的实例，因此`super()`相当于`A.prototype.constructor.call(this)`

```纯文本
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}

new A(); // A
new B(); // B
```


> 作为函数时，`super()`只能在子类的构造函数中调用


`super`作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类

```纯文本
class A {
  p() {
    return 2;
  }
}
class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}
const b = new B();
```


由于`super`指向父类的原型对象，所以定义在父类实例上的方法和属性，是无法通过`super`调用的

```纯文本
class A {
  constructor () {
    this.p = 2;
  }
}
class B extends A {
  get m() {
    return super.p;
  }
}
const b = new B();

b.m; // undefined
```


ES6 规定，在子类普通方法中通过`super`调用父类的方法时，方法内部的`this`指向当前子类的实例

```纯文本
class A {
  constructor() {
    this.x = 1;
  }

  print() {
    console.log(this.x);
  }
}
class B extends A {
  constructor() {
    super();
    this.x = 2;
  }

  m() {
    super.print();
  }
}
const b = new B();

b.m(); // 2
```


由于`this`指向子类实例，所以所以如果通过`super`对某个属性赋值，这是`super`就是`this`，赋值的属性会变成子类实例的属性

```纯文本
class A {
  constructor() {
    this.x = 1;
  }
}
class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}
const b = new B();

b.x; // 3
```


如果`super`作为对象，用在静态方法中，这时`super`指向父类，而不是父类的原型对象

```纯文本
class Parent {
  static myMethod = msg => console.log(`static ${msg}`);

  myMethod(msg) {
    console.log(`instance ${msg}`);
  }
}
class Child extends Parent {
  static myMethod = msg => super.myMethod(msg);

  myMethod(msg) {
    super.myMethod(msg);
  }
}
const child = new Child();

Child.myMethod(1); // static 1
child.myMethod(2); // instance 2
```


在子类静态方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类，而不是子类的实例

```纯文本
class A {
  constructor() {
    this.x = 1;
  }

  static print() {
    console.log(this.x);
  }
}
class B extends A {
  constructor() {
    super();
    this.x = 2;
  }

  static x = 3;

  static m() {
    super.print();
  }
}

B.m(); // 3
```


使用`super`时，必须显式指定是作为函数，还是作为对象使用

```纯文本
class A {}
class B extends A {
  constructor() {
    super();
    console.log(super); // Error
  }
}
```


对象总是继承其他对象，所以可以在任意一个对象中，使用`super`关键字

```纯文本
const obj = {
  toString() {
    return `MyObject: ${super.toString()}`;
  }
};

obj.toString(); // MyObject: [object Object]
```


prototype, **proto**

大多数浏览器的 ES5 实现之中，每个对象都有

`__proto__`

属性，指向对应的构造函数的

`prototype`

属性



Class 作为构造函数的语法糖，同时有

`prototype`

属性和

`__proto__`

属性，因此同时存在两条继承链

1. 子类的`__proto__`属性，表示构造函数的继承，总是指向父类
2. 子类的`prototype`属性的`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性<br />class A {}<br />class B extends A {}<br />B.**proto** === A; // true<br />B.prototype.**proto** = A.prototype; // true

因为类的继承是按照下面模式实现的

```纯文本
class A {}
class B {}

Object.setPrototypeOf(B.prototype, A.prototype);
Object.setPrototypeOf(B, A);

const b = new B();
```


作为一个对象，子类

`B`

的原型是父类

`A`



作为一个构造函数，子类

`B`

的原型是父类

`A`

原型对象的实例

```纯文本
B.prototype = Object.create(A.prototype);
// 等同于
B.prototype.__proto__ = A.prototype;
```


原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构

- Boolean
- Number
- String
- Array
- Object
- Function
- Date
- RegExp
- Error

ES5 规定，这些原生构造函数是无法继承的（子类无法获取原生构造函数的内部属性）



ES6 允许继承原生构造函数定义子类

```纯文本
class MyArray extends Array {
	constructor(..args) {
		super(..args);
	}
}
const arr = new MyArray();

arr[0] = 12;
arr.length; // 1
```


Mixin 模式的实现

Mixin 指的是多个对象合成一个对象，新对象具有各个组成成员的接口

```纯文本
const a = {
	a: 'a'
};
const b = {
	b: 'b'
};
const c = {...a, ...b}; // {a: 'a', b: 'b'}
```


下面是一个更完备的实现，将多个类的接口混入另一个类

```纯文本
function mix(...mixins) {
	class Mix {
		constructor() {
			for (const mixin of mixins) {
				copyProperties(this, new mixin());
			}
		}
	}

	for (const mixin of mixins) {
		copyProperties(Mix, mixin);
		copyProperties(Mix.prototype, mixin.prototype);
	}
	return Mix;
}

function copyProperties(target, source) {
	for (const ke of Reflect.ownKeys(source)) {
		if (key ! == 'constructor' && key !== 'prototype' && key !== 'name') {
			const desc = Object.getOwnPropertyDescriptor(source, key);

			Object.defineProperty(target, key, desc);
		}
	}
}

class DistributedEdit extends mix(Loggable, Serialzable) {}
```


