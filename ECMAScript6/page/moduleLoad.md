# moduleLoad

Module 的加载实现

浏览器加载

传统方法

HTML 网页中，浏览器通过`<script>`标签加载 JavaScript 脚本

```纯文本
<script type="application/javascript">
	// module code
</script>

<script type="application/javascript" src="path/module.js"></script>
```


由于浏览器默认脚本语言是 JavaScript，因此`type="application/javascript"`可以省略

默认情况下，浏览器是同步加载 JavaScript 脚本<br />渲染引擎遇到`<script>`标签就会停下来，等到执行完脚本，再继续向下执行<br />如果是外部脚本，还必须加入脚本下载的时间

如果脚本体积很大，下载和执行时间就会很长，因此造成浏览器堵塞<br />浏览器允许脚本异步加载

```纯文本
<script src="path/module.js" defer></script>

<script src="path/module.js" async></script>
```


渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接执行后面的操作

- `defer`要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行。多个`defer`脚本，会按照它们在页面中出现的顺序加载
- `async`一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。多个`async`脚本是不能保证加载顺序的

加载规则

浏览器加载 ES6 模块，也使用`<scirpt>`标签，但要加入`type="module"`属性

```纯文本
<script type="module" src="./foo.js"></script>
```


浏览器对带有`type="module"`的`<script>`，都是异步加载<br />等同于打开了`<script>`标签的`defer`属性

```纯文本
<script type="module" src="./foo.js"></script>
<!-- 等同于 -->
<script type="module" src="./foo.js" defer></script>
```


多个`<script type="module">`，它们会按照在页面中出现的顺序依次执行

`<script>`标签的`async`属性也可以打开，这时只要加载完，渲染引擎就会中断渲染立即执行

```纯文本
<script type="module" src="./foo.js" async></script>
```


ES6 模块也允许内嵌在网页中

```纯文本
<script type="module">
	import utils from './utils.js';

	// other code
</script>
```


对于外部的模块脚本，需要注意：

- 代码在模块作用域之中执行，而不是在全局作用域运行。模块内部的顶层变量，外部不可见
- 模块脚本自动采用严格模式
- 模块之中，可以使用`import`加载其他模块（`.js`后缀不可省略）。也可以使用`export`输出对外的接口
- 模块之中，顶层的`this`返回`undefined`，而不是`window`
- 同一个模块如果加载多次，将只执行一次<br />import utils from '[https://example.com/js/utils.js](https://example.com/js/utils.js)';<br />const x = 1;<br />x === window.x; // false<br />this === undefined; // true

ES6 模块与 CommonJS 模块的差异

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口

CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值

```纯文本
// lib.js
let counter = 3;
function inCounter() {
	counter++;
}

module.exports = {
	counter: counter,
	inCounter: inCounter
}

// main.js
const mod = require('./lib');

mod.counter; // 3
mod.inCounter();
mod.counter; // 3
```


ES6 模块是动态引用，并且不会换存值，模块里的变量绑定其所在的模块

```纯文本
// lib.js
export let counter = 3;
export function incCounter() {
	counter++;
}

// main.js
import {counter, incCounter} from './lib';

counter; // 3
incCounter();
counter; // 4
```


ES6 输入的模块变量是只读的，对它进行赋值会报错

```纯文本
// lib.js
export let obj = {};

// main.js
import {obj} from './lib';

obj.prop = 123; // OK
obj = {}; // Error
```


`export`通过接口输出的是同一个值。不同的脚本加载这个接口，得到的都是同一个实例

```纯文本
// mod.js
function C() {
	this.sum = 0;
	this.add = function () {
		this.sum += 1;
	};
	this.show = function () {
		console.log(this.sum);
	}
}

export let c = new C();

// x.js
import {c} from '/mod';

c.add();

// y.js
import {c} from './mod';
c.show();

// main.js
import './x';
import './y';
```


Node.js 加载

Node.js 对 ES6 模块的处理比较麻烦，因为它有自己的 CommonJS 模块，与 ES6 模块格式是不兼容的<br />目前的解决方案，是将两者分开，ES6 模块和 CommonJS 模块采用各自的加载方案

Node.js 要求 ES6 模块采用`.mjs`后缀文件名<br />如果不希望后缀名改为`.mjs`，可以在项目的`package.json`文件指定`type`字段为`module`

```纯文本
{
	"type": "module"
}
```


如果这时还要使用 CommonJS 模块，那么需要将 CommonJS 模块脚本的后缀名改为`.cjs`

> ES6 模块和 CommonJS 模块尽量不要混用


main

`package.json`文件有两个字段可以知道文件的入口文件：`main`和`exports`<br />比较简单的模块，可以只使用`main`字段，指定模块加载的入口文件

```纯文本
// ./node_mdoules/es-module-package/package.json
{
	"type": "module",
	"main": './src/index.js'
}
```


exports

`exports`字段的优先级高于`main`字段

1. 子目录别名

可以指定脚本或子目录的别名

```纯文本
// ./node_mdoules/es-module-package/package.json
{
	"exports": {
		"./submodule": "./src/submodule.js"
	}
}
```


上面代码指定`src/submodule.js`别名为`submodule`，然后就可以从别名加载和这个文件

```纯文本
import submodule from 'es-module-package/submodule'
```


1. main 的别名

`exports`字段的别名如果是`.`，就代表模块的主入口，优先级高于`main`字段，并且可以直接简写成`exports`字段

```纯文本
{
	"exports": {
		".": "./main.js"
	}
}
// 等同于
{
	"exports": "./main.js"
}
```


1. 条件加载

利用`.`别名，可以为 ES6 模块和 CommonJS 模块指定不同的入口

```纯文本
{
	"type": "module",
	"exports": {
		".": {
			"require": "./main.cjs",
			"default": "./main.js"
		}
	}
}
```


ES6 模块加载 CommonJS 模块

`package.json`文件的`main`字段指定`CommonJS`入口，给 Node.js 使用<br />`module`字段指定 ES6 模块入口，给打包工具使用

```纯文本
// ./node_modules/pkg/package.json
{
	"type": "module",
	"main": "./index.cjs",
	"exports": {
		"require": "./index.cjs",
		"default": "./wrapper.mjs"
	}
}
```


CommonJS 模块加载 ES6 模块

CommonJS 的`reuqire`命令不能加载 ES6 模块

Node.js 的内置模块

Node.js 的内置模块可以整体加载，也可以加载指定的输出项

```纯文本
// 整体加载
import EventEmitter from 'events';

const e = new EventEmitter();

// 加载指定的输出项
import {readFile} from 'fs';
```


加载路径

ES6 模块的加载路径必须给出脚本的完整路径，不能省略后缀名

内部变量

ES6 模块应该是通用的，同一个模块不用修改，就可以用在浏览器环境和服务器环境<br />Node 规定 ES6 模块之中不能使用 CommonJS 模块特有的一些内部变量

- this
- arguments
- reuqire
- module
- exports
- __filename
- __dirname

循环加载

“循环加载”指的是，`a`脚本的执行依赖`b`脚本，`b`脚本的执行又依赖`a`脚本

```纯文本
// a.js
var b = require('b');

// b.js
var a = require('a');
```


CommonJS 模块的加载原理

CommonJS 的一个模块，就是一个脚本文件<br />`reuqire`命令命令第一次加载该脚本，就会执行整个脚本，然后在内存中生成一个对象

```纯文本
{
	id: '...',
	exports: {...},
	loaded: true,
	...
}
```


- id：模块名
- exports: 模块输出的各个接口
- loaded：布尔值，表示该模块的脚本是否执行完毕

以后需要用到这个模块的时候，就会到`exports`属性上面取值

CommonJS 模块的循环加载

CommonJS 模块的重要特性是加载时执行，即脚本代码在`require`的时候，就会全部执行<br />一旦出现某个模块被“循环加载”，就只输出已经执行的部分，还未执行的部分不会输出

```纯文本
// a.js
exports.done = false;
var b = require('./b.js');

console.log(`在 a.js 当中，b.done = ${b.done}`);
exports.done = true;
console.log(` a.js 执行完毕`);

// b.js
exports.done = false;
var a = reuqire('./a.js');

console.log(`在 b.js 当中，a.done = ${a.done}`);
exports.done = true;
console.log('b.js 执行完毕');

// main.js
var a = require('./a.js');
var b = reuqire('./b.js');

console.log(`在 main.js 当中，a.done = ${a.done}，b.done = ${b.done}`);
// 在 b.js 当中，a.done = false
// b.js 执行完毕
// 在 a.js 当中，b.done = true
// a.js 执行完毕
// 在 main.js 当中，a.done = true, b.done = true
```


ES6 模块的循环加载

```纯文本
// a.mjs
import {bar} from './b';

console.log('a.mjs');
console.log(bar);

export let foo = 'foo';

// b.mjs
import {foo} from './a';

console.log('b.mjs');
console.log(foo);

export let bar = 'bar';

$ node a.mjs
// b.mjs
// ReferenceError: foo is not defined
```


