# generatorSync

Generator 函数的异步应用

异步编程

所谓“异步”，简单说就是一个任务不是连续完成的，可以理解成任务被分为两个阶段。先执行第一阶段，然后转而执行其他任务，等做好了准备，再回过头执行第二阶段

相应地，连续执行就叫作同步

ES6 诞生之前，异步编程的方式

- 回调函数
- 事件监听
- 发布/订阅
- Promise

回调函数

JavaScript 语言对异步编程的实现，就是回调函数



所谓回调函数，就是把任务的第二阶段单独写在函数里，等到重新执行这个任务的时候，就直接调用这个函数

```纯文本
fs.readFIle('/etc/passwd', 'utf-8', function (err, data){
	if (err){
		throw err;
	}
	console.log(data);
});
```


Promise

回调函数本身没有问题，它的问题出现在多个回调函数嵌套

```纯文本
fs.readFile(fileA, 'utf-8', function (err, data) {
	fs.readFile(fileB, 'utf-8', function (err, data) {

	});
});
```


Promise 对象就是为了解决这个问题而提出的。



它不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用

```纯文本
const readFile = require('fs-readfile-promise');

readFile(fileA)
	.then(data => console.log(data.toString()))
	.then(() => readFile(fileB))
	.then(data => console.log(data.toString()))
	.catch(err => console.log(err));
```


Promise 最大的问题就是代码冗余

Generator 函数

协程

传统的编程语言，早有异步编程的解决方案。其中有一种叫做“协程”，意思是多个线程互相协作，完成异步任务

运行流程

1. 协程`A`开始执行
2. 协程`A`执行到一半，进入暂停，执行权转移到协程`B`
3. 协程`B`交还执行权
4. 协程`A`恢复执行<br />function* asyncJob() {<br />const f = yield readFile(fileA);<br />}

协程遇到`yield`命令就暂停，等到执行权返回，再从暂停的地方继续往后执行

协程的 Generator 函数实现

Generator 函数是协程在 ES6 的实现，最大特点就是可以交出函数的执行权

```纯文本
function* gen(x) {
	const y = yield x + 2;
	return y;
}
const g = gen(1);

g.next();
g.next();
```


Generator 函数的数据交换和错误处理

Generator 函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因



`next`

返回值的

`value`

属性，是 Generator 函数向外输出数据



`next`

方法还可以接受参数，向 Generator 函数体内输入数据

```纯文本
function* gen(x) {
	const y = yield x + 2;
	return y;
}
const g = gen(1);

g.next(); // {value: 3, done: false}
g.next(2); // {value: 2, done: true}
```


Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误

```纯文本
function* gen(x) {
	try {
		const y = yield x + 2;
	} catch(e) {
		console.log(e);
	}
	return y;
}
const g = gen(1);

g.next();
g.throw('failed'); // 'failed'
```


异步任务封装

执行一个真实的任务

```纯文本
const fetch = require('node-fetch');

function* gen(){
	const url = 'https://api';
	const result = yield fetch(url);

	console.log(result.bio);
}
const g = gen();
const result = g.next();

result.value
	.then(data => data.json())
	.then(data => g.next(data));
```


Thunk

Thunk 函数是自动执行 Generator 函数的一种方法

参数的求值策略

函数的参数到底何时求值

1. 传值调用：在进入函数体之前进行求值
2. 传名调用：在函数体内用到的时候求值

传值调用，比较简单，但有可能造成性能损失

Thunk 函数的含义

编译器的“传名调用”实现，往往是将参数放到一个临时函数当中，再将这个临时函数传入函数体。这个临时函数就叫作 Thunk 函数

```纯文本
function f(m) {
	return m * 2;
}
f(x + 5);

// 等同于
const thunk = function () {
	return x + 5;
};
function f(thunk) {
	return thunk() * 2;
}
```


这就是 Thunk 函数的定义，它是“传名调用”的一种实现策略，用来替换某个表达式

JavaScript 语言的 Thunk 函数

JavaScript 语言是传值调用



Thunk 函数替换的不是表达式，而是多参数函数



将其替换成一个只接受回调函数作为参数的单参数函数

```纯文本
// 正常版
fs.readFile(fileName, callback);

// Thunk 版
const Thunk = function (fileName) {
	return function (callback) {
		return fs.readFile(fileName, callback);
	};
};
const readFileThunk = Thunk(fileName);

readFileThunk(callback);
```


任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式

```纯文本
const Thunk = fn => {
	return args => {
		return callback => {
			return fn.call(this, ...args, callback);
		}
	}
};
const readFileThunk = Thunk(fs.readFile);

readFileThunk(fileA)(callback);
```


Thunkify 模块

生产环境的转换器，建议使用 Thunkify 模块

```纯文本
$ npm install thunkify
```


使用方式如下

```纯文本
const Thunkify = require('thunkify');
const fs = require('fs');
const read = thunkify(fs.readFile);

read('package.json')(function (err, str) {});
```


Generator 函数的流程管理

Thunk 函数可以用于 Generator 函数的自动流程管理

```纯文本
function* gen() {}
const g = gen();
let res = g.next();

while(!res.done) {
	console.log(res.value);
	res = g.next();
}
```


上面代码中，Generator 函数`gen`会自动执行完所有步骤。但是，这不适合异步操作

```纯文本
const fs = require('fs');
const thunkify = require('thunkify');
const readFileThunk = thunkify(fs.readFile);
const gen = function* () {
	const r1 = yield readFileThunk('/etc/fstab');
	console.log(r1.toString());
	const r2 = yield readFileThunk('/etc/shells');
	console.log(r2.toString());
};
const g = gen();
const r1 = g.next();

r1.value(function(err, data) {
	if (err) trhow err;
	const r2 = g.next(data);
	r2.value(function (err, data) {
		if (err) throw err;
		g.next(data);
	})
});
```


Thunk 函数的自动流程管理

基于 Thunk 函数的 Generator 执行器

```纯文本
function run(fn) {
	const gen = fn();

	function next(err, data) {
		const result = gen.next(data);

		if (result.done){
			return;
		}
		result.value(next);
	}
	next();
}
function* g() {}
run(g);
```


使用该 Generator 执行器的前提是，每个异步操作都是 Thunk 函数

```纯文本
const g = function* () {
	const f1 = yield readFileThunk('fileA');
	const f2 = yield readFileThunk('fileB');
	//...
	const fn = yiled readFileThunk('fileN');
};

run(g);
```


co 模块

co 模块用于 Generator 函数的自动执行

```纯文本
const co = require('co');
const gen = function* () {
	const f1 = yield readFile('/etc/fstab');
	const f2 = yield readFile('/etc/shells');

	console.log(f1.toString());
	console.log(f2.toString());
};

co(gen);
```


基于 Promise 对象的自动执行

```纯文本
const fs = require('fs');
const readFile = fileName => {
	return new Promise((resolve, reject) => {
		fs.readFile(fileName, (err, data) => {
			if (err){
				return reject(error);
			}
			resolve(data)
		});
	});
};
const gen = function* () {
	const f1 = yield readFile('/etc/fstab');
	const f2 = yield readFile('/etc/shells');

	console.log(f1.toString());
	console.log(f2.toString());
};
const g = gen();

g
	.next()
	.value
	.then(data => g.next(data).value)
	.then(data => g.next(data));
```


