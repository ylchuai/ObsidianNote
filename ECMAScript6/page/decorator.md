# decorator

装饰器

装饰器是一种与类相关的语法，用来注释或修改类和类的方法<br />装饰器是一种函数，写成`@+函数名`<br />它可以放在类和类方法的定义前面

```纯文本
@frozen
class Foo {
	@configurable(false)
	@enumerable(true)
	method() {}

	@throttle(500)
	expensiveMethod() {}
}
```


类的装饰器

装饰器可以用来装饰整个类

```纯文本
@testable
class MyTestableClass {}

function testable(target) {
	target.isTestable = true;
}
MyTestableClass.isTestable; // true
```


基本上，装饰器的行为就是下面这样

```纯文本
@decorator
class A {}

// 等同于
A = decorator(A) || A;
```


装饰器是一个对类进行处理的函数<br />装饰器函数的第一个参数，就是索要装饰的目标类

```纯文本
function testable(target) {}
```


如果一个参数不够用，可以在装饰器函数外面再封装一层函数

```纯文本
function testable(isTestable) {
	return function (target) {
		target.isTestable = isTestable;
	}
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable; // true

@testable(false)
class MyClass {}
MyClass.isTestable; // false
```


> 装饰器对类的行为的改变，是代码编译时发生的


如果想要添加实例属性，可以通过目标类的`prototype`对象操作

```纯文本
function testable(target) {
	target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();

obj.isTestable; // true
```


下面是另一个例子

```纯文本
// mixins.js
export function mixins(..list) {
	return function (target) {
		Object.assign(target.prototype, ...list);
	}
}

// main.js
import {mixins} from './mixins';

const Foo = {
	foo() {
		console.log('foo');
	}
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();

obj.foo(); // 'foo'
```


实际开发中，React 与 Redux 库结合使用时

```纯文本
class MyReactComponent extends React.Component {}
export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);

// 装饰器写法
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```


方法的装饰器

装饰器不仅可以修饰类，还可以装饰类的属性

```纯文本
class Person {
	@readonly
	name () {
		return `${this.first} ${this.last}`;
	}
}
```


装饰器函数`readonly`一共可以接受三个参数

```纯文本
function readonly(target, name, descriptor) {
	descriptor.writable = false;
	return descriptor;
}
readonly(Person.prototype, 'name', descriptor);
```


装饰器第一个参数是类的原型对象，装饰器的本意是要装饰类的实例，这个时候实例还没有生成，所以只能去装饰原型<br />第二个参数是所要装饰的属性名<br />第三个参数是该属性的描述对象

下面是`@log`装饰器，可以起到输入日志的作用

```纯文本
class Math {
	@log
	add(a, b) {
		return a + b;
	}
}
function log(target, name, descriptor) {
	const oldValue = descriptor.value;

	descriptor.value = function () {
		console.log(`Calling ${name} with`, argyments);
		return oldValue.apply(this, arguments);
	}
	return descriptor;
}

const math = new Math();

math.add(2, 4);
```


装饰器有注释的作用

```纯文本
@testable
class Person {
	@readonly
	@nonenumberable
	name () {
		return `${this.first} ${this.last}`;
	}
}
```


下面使用装饰器写的组件

```纯文本
@Component({
	tag: 'my-component',
	styleUrl: 'my-component.scss'
})
export class MyComponent {
	@Prop() first: string;
	@Prop() last: string;
	@State() isVisible: boolean = true;

	render() {
		return (
			<p>Hello, my name is {this.first} ${this.last}</p>
		);
	}
}
```


如果同一个方法有多个装饰器，先从外到内进入，然后由内向外执行

```纯文本
function dec(id) {
	console.log('evaluated', id);
	return (target, property, descriptor) => console.log('executed', id);
}
class Example {
	@dec(1)
	@dec(2)
	method () {}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```


为什么装饰器不能用于函数

因为存在函数提升

