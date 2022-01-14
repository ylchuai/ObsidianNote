# promise

Promise 对象

含义

Promise 是异步编程的一种解决方案，比传统的解决方案“回调函数和事件”更合理更强大。Promise 对象有以下两个特点

1. 对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，才可以决定当前是哪一种状态
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise 状态的改变，只有两种可能：从 pending 变为 fulfilled 和从 pending 变为 rejected

有了 Promise 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数

基本用法

Promise 对象是一个构造函数，用来生成 Promise 实例

```纯文本
const promise = new Promise(function (resolve, reject){
	if (true){
		resolve(value);
	} else {
		reject(error);
	}
});
```


`Promise` 构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`，他们是两个函数，由 JavaScript 引擎提供，不用自己部署

`resolve`函数的作用是，将 Promise 对象的状态从 pending（进行中）变为 reoslved（成功）。在异步操作成功时调用，并将异步操作的结果，作为参数传递出去

`reject`函数的作用是，将 Promise 对象的状态从 pending（进行中）变为 rejected（失败）。在异步操作失败时调用，并将异步操作报出来的错误，作为参数传递出去

```纯文本
function timeout(ms) {
	return new Promise((resolve, reject) => {
		setTimeout(resolve, ms, 'done');
	});
}
timeout(100)
	.then(value => {
		console.log(value);
	});
```


Promise 新建后就会立即执行

```纯文本
const promise = new Promise(function (resolve, reject) {
	console.log('Promise');
	resolve();
});

promise
	.then(function () {
		console.log('resolved')	;
	});
console.log('Hi');
// Promise
// Hi
// resolved
```


下面时异步加载图片的例子

```纯文本
function loadImageAsync(url) {
	return new Promise(function (resolve, reject) {
		const image = new Image();

		image.onload = function () {
			resolve(image);
		};
		image.onerror = function () {
			reject(new Error('failed'));
		};
	});
}
```


下面是一个 Promise 对象实现 Ajax 操作的例子

```纯文本
const getJSON = function () {
	const promise = new Promise(function (resolve, reject) {
		const handler = function () {
			if (this.readyState !== 4){
				return;
			}
			if (this.state === 200){
				resolve(this.response);
			} else {
				reject(new Error(this.statusText));
			}
		};
		const client = new XMLHttpRequest();

		client.open('GET', url);
		client.onreadystatechange = handler;
		client.responseType = 'json';
		client.setRequestHeader('Accept', 'application/json');
		client.send();
	});

	return promise;
};

getJSON('/posts.json')
	.then(function (json) {
		console.log(`Contents: ${json}`);
	}, function (error) {
		console.error(`failed: ${error}`);
	});
```


如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给回调函数

`resolve`函数的参数除了正常的值以外，还可以是另一个 Promise 实例

```纯文本
const p1 = new Promise(function (resolve, reject) {});
const p2 = new Promise(function (resolve, reject) {
	resolve(p1);
});
```


注意，这时`p1`的状态就会传递给`p2`，也就是说，`p1`的状态决定了`p2`的状态。<br />如果，`p1`的状态是`peding`，那么`p2`的回调函数就会等待`p1`的状态该拜年<br />如果，`p1`的状态已经是`resolved`或者`rejected`，那么`p2`的回调函数就会立即执行

```纯文本
const p1 = new Promise(function (resolve, reject) {
	setTimeout(() => reject(new Error('fail')), 3000);
});
const p2 = new Promise(function (resolve, reject) {
	setTimeout(() => resolve(p1)， 1000);
});

p2.
	.then(result => console.log(result))
	.catch(error => console.log(error));
// Error: fail
```


注意，调用`resolve`或者`reject`并不会终结 Promise 的参数函数的执行

```纯文本
new Promise((resolve, reject) => {
	resolve(1);
	console.log(2)
}).then(r => {
	console.log(r);
});
// 2
// 1
```


一般来说，调用`resolve`或`reject`以后，Promise 的使命就完成了，后继操作应该放到`then`方法里面。所以，最好在它们前面加上`return`语句

```纯文本
new Promise((resolve, reject) => {
	return resolve(1);
});
```


Promise.prototype.then()

Promise 实例具有

`then`

方法，作用是为 Promise 实例添加状态改变时的回调<br />

`then`

方法的第一个参数是

`resolved`

状态的回调函数，第二个参数（可选）是

`rejected`

状态的回调函数



`then`

方法返回的是一个新的 Promise 实例（不是原来那个 Promise 实例）。因此可以采用链式语法

```纯文本
getJSON('/posts.json')
	.then(json => json.post)
	.then(post => console.log(post));
```


采用链式的`then`，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个 Promise 对象，这时后一个回调函数，就会等待该 Promise 对象的状态发生变化，才会调用

```纯文本
getJSON('/posts.json')
	.then(post => {
		return getJSON(post.commentURL);
	})
	.then(comments => {
		console.log(`resolved: ${comments}`);
	}, err => {
		console.log(`rejected: ${err}`);
	});
```


Promise.prototype.catch()

`Promise.prototype.catch`方法是`.then(null, rejection)`或`.then(undefined, rejection)`的别名，用于指定发生错误时的回调函数

```纯文本
getJSON('/posts.json')
	.then(posts => {})
	.catch(err => {
		console.log(`failed: ${err}`);
	});
```


另外，`then`方法指定的回调函数，如果运行中抛出错误，也会被`catch`方法捕获

```纯文本
p
	.then(val => console.log(`fulfilled: ${val}`))
	.catch(err => console.log(`rejected: ${err}`));
// 等同于
p
	.then(val => console.log(`fulfilled: val`))
	.then(null, err => console.log(`rejected: ${err}`));
```


如果 Promise 状态已经变成`resolved`，再抛出错误时无效的

```纯文本
const promise = new Promise((resolve, reject) => {
	resolve('ok');
	throw new Error('test');
});

promise
	.then(val => console.log(val))
	.catch(err => console.log(err));
```


> 因为 Promise 的状态一旦改变，就永久保存该状态，不会再变了


Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止

```纯文本
getJSON('/posts.json')
	.then(post => getJSON(post.commentURL))
	.then(comments => {})
	.catch(err => {
		// 处理前面三个 Promise 产生的错误	
	});
```


一般来说，不要在`then`方法里面定义`rejected`状态的回调函数（即`then`的第二个参数），总是使用`catch`方法

```纯文本
// bad
promise
	.then(data => {}, err => {});

// good
promise
	.then(data => {})
	.catch(err => {});
```


如果没有使用`catch`方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码

```纯文本
const promise = function () {
	return new Promise((resolve, reject) => {
		resolve(x + 2);
	});
};

promise
	.then(() => console.log('ok'));
setTimeout(() => console.log(123), 2000);
// ReferenceError: x is not defined
// 123
```


`catch`方法返回的还是一个 Promise 对象，因此后面还可以接着调用`then`方法

```纯文本
const promise = function () {
	return new Promise((rs, rj) => rs(x + 2));
};

promise
	.catch(err => console.log('oh no', err))
	.then(() => console.log('carry on'));
// oh no [ReferenceError: x is not defined]
// carry on
```


Promise.prototype.finally()

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作

```纯文本
promise
	.then(data => {})
	.catch(err => {})
	.finally(() => {});
```


下面是一个例子，服务器使用 Promise 处理请求，然后使用`finally`方法关闭服务器

```纯文本
server.listen(prot)
	.then(() => {})
	.finally(() => server.stop);
```


Promise.all()

`Promise.all`

方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。



接受一个数组（具有 Iterator 接口的数据）作为参数，数组的成员都是 Promise 实例

```纯文本
const p = Promise.all([p1, p2, p3]);
```


`Promise.all()`实例的状态由成员状态决定：

1. 只有所有成员的状态变成`fulfilled`，`Promise.all()`实例的状态才会变成`fulfilled`。此时，成员的返回值组成一个数组，传递给`Promise.all()`实例的回调函数
2. 只要成员中有一个状态变成`rejected`，`Promise.all()`实例的状态就会变成`rejected`。此时，成员第一个变成`rejected`的实例的返回值，会传递给`Promise.all()`实例的回调函数<br />const promises = [2, 3, 5, 7, 11 , 13].map(id => getJSON(`/posts/${id}.json`));<br />promise.all(promises)<br />.then(post => {})<br />.catch(err => {});

如果，作为参数的 Promise 实例，自己定义了`catch`方法，那么它一旦被`rejected`，并不会触发`Promise.all()`的`catch`方法

```纯文本
const p1 = new Promise((resolve, reject) => {
	resolve('hello');
}).then(r => r)
	.catch(e => e);
const p2 = new Promise((resolve, reject) => {
	throw new Error('failed');
}).then(r => r)
	.catch(e => e);

Promise.all([p1, p2])
	.then(r => console.log(r))
	.catch(e => console.log(e))
// ['hello', Error: failed]
```


Promise.race()

`Promise.race`

方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。



接受一个数组（具有 Iterator 接口的数据）作为参数，数组的成员都是 Promise 实例。



只要成员之中有一个实例率先改变状态，

`Promise.race()`

实例的状态就会改变。



率先改变的 Promise 实例的返回值，就会传递给

`Promise.all()`

实例的回调函数

```纯文本
const p = Promise.race([
	fetch(url),
	new Promise((resolve, reject) => {
		setTimeout(() => reject(new Error('request timeout')), 5000)	
	})
]);

p
	.then(data => {})
	.then(err => {});
```


Promise.allSettled()

`Promise.allSettled`

方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例



只有等到这些参数实例都返回结果，不管是

`fulfilled`

还是

`rejected`

，包装实例才会结束



返回值为新的 Promise 实例，一旦结束，状态总是

`fulfilled`

，不会变成

`rejected`

<br />

`Promise.allSettled()`

实例的监听函数接受到参数是一个数组，每个成员是 Promise 实例的返回值

```纯文本
const p1 = Promise.resolve(42);
const p2 = Promise.reject(-1);
const p = Promise.allSettled([p1, p2]);

p.then(r => console.log(r));
//	[
//		{status: 'fulfilled', value: 42},
//		{status: 'rejected', reason: -1}
//	]
```


下面时返回值用法的例子

```纯文本
const ps = [
	fetch(url1),
	fetch(url2)
];
const results = await Promise.allSettled(ps);
const fulfilledPromise = results.filter(p => p.status === 'fulfilled');
const rejectedPromise = results
	.filter(p => p.status === 'rejected')
	.map(p => p.reason);
```


Promise.resolve()

将现有对象转化为 Promise 对象

```纯文本
Promise.resolve($.ajax(url));

Promise.resolve('foo');
// 等同于
new Promise(resolve => resolve('foo'));
```


`Promise.resolve`方法的参数分为四种情况

- Promise 实例

那么 `Promise.resolve()`将不做任何修改，直接返回这个实例

- thenable 对象

thenable 对象指的是具有`then`方法的对象

```纯文本
const thenable = {
	then(resolve, reject) {
		resolve(42);
	}
};
```


`Promise.resolve`方法会将这个对象转化为 Promise 对象，然后立即执行 thenable 对象的`then`方法

```纯文本
const thenable = {
	then(resolve, reject) {
		resolve(42);
	}
};
const p = Promise.resolve(thenable);

p.then(r => console.log(r));
// 42
```


- 不具有`then`方法的对象，或根本不是对象

如果参数是一个原始值，则`Promise.resolve`方法返回一个新的 Promise 对象，状态为`resolved`

```纯文本
const p = Promise.resolve('hello');

p.then(r => console.log(r));
// hello
```


- 不带有任何参数

直接返回一个`resolved`状态的 Promise 对象

```纯文本
const p = Promise.resolve();

p.then(() => {});
```


Promise.reject()

`Promise.reject(reason)`方法也会返回一个新的 Promise 实例，该实例的状态为`rejected`

```纯文本
const p = Promise.reject('failed');
// 等同于
const p = new Promise((resolve, reject) => reject('failed'));

p.then(null, e => console.log(e));
// failed
```


参数会原封不动地作为`reject`理由

```纯文本
const thenable = {
	then(resolve, reject) {
		reject('failed');
	}
};

Promise
	.reject(thenable)
	.catch(e => console.log(e === thenable));
// true
```


