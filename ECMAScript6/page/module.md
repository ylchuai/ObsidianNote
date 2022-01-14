# module

Module 的语法

概述

历史上 JavaScript 一直没有模块体系



ES6 之前，社区制定了一些模块加载方案，主要是 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器



ES6 在语言标准的层面，实现了模块功能，成为浏览器和服务器通用的模块解决方案



ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量



CommonJS 和 AMD 模块，都只能在运行时确定关系

```纯文本
const {stat, exists, readFile} = require('fs');
// 等同于
const _fs = require('fs');
const stat = _fs.stat;
const exists = _fs.exists;
const readFile = _fs.readFile;
```


ES6 模块不是对象，而是通过`export`命令显式指定输出的代码，再通过`import`命令输入

```纯文本
iimport {stat, exists, readFile} from 'fs';
```


由于 ES6 模块是编译时加载，使得静态分析成为可能

除了静态加载带来的各种好处，ES6 模块还有以下好处

- 不再需要加载`UMD`模块格式了，将来浏览器和服务器都会支持 ES6 模块
- 将来浏览器的新 API 就能用模块格式提供，不再需要做成全局变量
- 不再需要对象作为命名空间

严格模式

ES6 模块自动采用严格模式

严格模式主要由以下限制

- 变量必须声明后使用
- 函数的参数不能有同名属性
- 不能使用`with`语句
- 不能对只读属性赋值
- 不能使用 0 前缀表示八进制
- 不能删除不可删除的属性
- 不能删除变量`delete prop`，只能删除`delete global[prop]`
- `eval`不会在它的外层作用域引入变量
- `eval`和`arguments`不能被重新赋值
- `arguments`不会自动反应函数参数的变化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全局对象
- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
- 增加了保留字（`protected`，`static`和`interface`）

export

`export`命令用于规定模块的对外接口

一个模块就是一个独立文件，该文件内部的所有变量，外部无法获取



如果希望外部能读取模块内部的某个变量，就必须使用

`export`

关键字输出该变量

```纯文本
export const firstName = 'Michael';
export const lastName = 'Jackson';
export const year = 1958;
```


`export`的写法，除了上面这样，还有另外一种

```纯文本
const firstName = 'Michael';
const lastName = 'Jackson';
const year = 1958;

export {firstName, lastName, year};
```


`export`命令除了输出变量，还可以输出函数或者类

```纯文本
export function multiply(x, y) {
	return x * y;
}
```


通常情况下，`export`输出的变量就是本来的名字，但可以使用`as`关键字重命名

```纯文本
function v1() {}
function v2() {}

export {
	v1 as streamV1,
	v2 as streamV2,
	v2 as streamLatestVersion
};
```


`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应的关系

```纯文本
export 1； // Error

const m = 1;

export m; // Error
```


> 上面两种写法都是直接输出`1`。`1`只是一个值，不是接口


同样的，`function`和`class`的输出，也必须遵守这样的写法

```纯文本
function f() {}
export f; // Error

export function f() {};

function f() {}
export {f};
```


`export`输出的接口，与其对应的值是动态绑定关系

```纯文本
export let foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```


> 这一点与 CommonJS 规范完全不同。CommonJS	模块输出的是值的缓存


`export`命令可以出现在模块的任何位置，只要处于模块顶层就可以

```纯文本
function foo() {
	export default 'bar'; // Error
}
foo();
```


import

`import`命令用于输入其他模块提供的功能

```纯文本
import {firstName, lastName, year} from './profile.js';

function setName(element) {
	element.textContent = `${firstName} ${lastName} ${year}`;
}
```


> 大括号里面的变量名，必须与被倒入模块对外接口的名称相同


`import`命令可以使用`as`关键字对输入的变量重命名

```纯文本
import {lastName as surname} from './profile.js';
```


`import`命令输入的变量都是只读的，因为它的本质是输入接口

```纯文本
import {a} from './xxx.js';

a = {}; // Error
```


但是如果`a`是一个对象，改写`a`的属性是可以的

```纯文本
import {a} from './xxx.js';

a.foo = 'hello';
```


> 上面代码中，`a`的属性可以改写成功，并且其他模块也可以读到改写后的值


`import`

后面的

`from`

指定模块文件的位置，可以是相对路径，也可以是绝对路径，

`.js`

后缀可以省略



如果只是模块名，那么必须有配置文件，告诉 JavaScript 引擎该模块的位置

```纯文本
import {myMethod} from 'util';
```


`import`命令具有提升效果，会提升到整个模块的头部，首先执行

```纯文本
foo();
import {foo} from 'my_module';
```


> 因为`import`命令是编译阶段执行的，在代码运行之前


由于`import`是静态执行的，所以不能使用表达式和变量，这些只有在运行时才能得到结果

```纯文本
import {'f' + 'oo'} from 'my_module'; // Error

const module = 'my_module';
import {foo} from module; // Error

// Error
if (x === 1){
	import {foo} from 'module1';
} else {
	import {foo} from 'module2';
}
```


> 在静态分析阶段，这些语法都没得到值


`import`语句会执行所加载的模块

```纯文本
import 'loadsh';
```


> 仅仅执行`loadsh`模块，但不输入任何值


如果多次重复执行同一句`import`语句，那么只会执行一次

```纯文本
import 'loadsh';
import 'loadsh';
```


模块的整体加载

除了指定加载某个输出值，还可以用整体加载，即用`*`指定一个对象，所有输出值都加载在这个对象上

```纯文本
// circle.js
export function area(radius) {
	return Math.PI * radius * radius;
}

export function circumFerence(radius) {
	return 2 * Math.PI * radius;
}

// main.js
import * as circle from './circle';

console.log(circle.area(4));
console.log(circle.cirumference(14));
```


模块整体加载所在的那个对象，应该是可以静态分析的，所以不允许运行时改变

```纯文本
import * as circle from './circle';

circle.foo = 'hello'; // Error
```


export default

为模块指定默认输出

```纯文本
// export-default.js
export default function () {
	console.log('foo');
}
```


其他模块加载该模块的时候，`import`命令可以为输出指定任意名字

```纯文本
// import-default.js
import customName from './export-default';

customName(); // 'foo'
```


> 显然，一个模块只能有一个默认输出，因此`export defualt`命令只能用一次


本质上，`export default`就是输出一个叫`default`的变量，然后系统允许你为他取任意名字

```纯文本
// modules.js
function add(x, y) {
	return x * y;
}
export {add as default};
// 等同于
export default add;

// app.js
import {default as foo} from 'modules';
// 等同于
import foo from 'modules';
```


正因为`export default`命令其实只是输出一个叫做`defualt`的变量，所以它后面不能跟变量声明语句

```纯文本
const a = 1;
export default a;

export default const a = 1; // Error
```


可以在一条`import`语句中，同时输入默认接口和其他接口

```纯文本
// modules.js
export default function(obj) {}
export function each() {}
export {each as forEach};

// main.js
import _, {each, forEach} from './modules';
```


`export default`也可以用来输出类

```纯文本
// MyClass.js
export default class {}

// main.js
import MyClass from './MyClass';

const obj = new MyClass();
```


export 和 import 的复合写法

如果在一个模块之中，先输入后输出同一个模块，`import`和`export`语句可以写在一起

```纯文本
export {foo, bar} from 'my_module';
// 等同于
import {foo, bar} from 'my_module';
export {foo, bar};
```


> 写在一起后，`foo`和`bar`实际上并没有被导入当前模块，只相当于对外转发了这两个接口，当值当前模块不能直接使用`foo`和`bar`


也支持接口重命名

```纯文本
export {foo as myFoo} from 'my_module';

export * from 'my_module';
```


具体接口改为默认接口

```纯文本
export {es6 as default} from './modules';
// 等同于
import {es6} from './modules';
export default es6;
```


默认接口改为具名接口

```纯文本
export {default as es6} from './modules';
```


> `export *`会忽略模块的`default`方法


