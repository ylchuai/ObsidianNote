# string

字符串的扩展

字符的 Unicode 表示法

ES6 加强了对 Unicode 的支持，允许采用`\uxxxx`形式表示一个字符，其中`xxxx`表示字符的 Unicode 码点

```纯文本
'\u0061'
// "a"
```


但是，这种表示法只限于码点在`\u0000`~`\uFFFF`之间的字符。超出整个范围的字符，必须使用两个双字节的形式表示。

```纯文本
'\uD842\uDFB7'
// "𠮷"

'\u20BB7'
"₻7"
```


ES6 对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符

```纯文本
'\u{20BB7}'
// "𠮷"
```


有了这种表示方法之后，JavaScript 共有 6 种方法可以表示一个字符

```纯文本
'\z'
'\172'
'\x7A'
'\u007A'
'\u{7A}'
// "z"
```


字符串的遍历器接口

ES6 为字符串添加了遍历器接口（Iterator），使得字符串可以被`for...of`遍历循环

```纯文本
for (let codePoint of 'foo'){
	console.log(codePoint)
}
// "f"
// "o"
// "o"
```


除了遍历字符串，这个遍历器最大的优点是可以识别大于`0xFFFF`的码点，传统的`for`循环无法识别这样的码点

```纯文本
let text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i += 1){
	console.log(text[i]);
}
// " "
// " "

for (let i of text){
	console.log(i);
}
// "𠮷"
```


`JSON.stringify()`的改造

根据标准，JSON 数据必须是 UTF-8 编码。但是现在的`JSON.stringify()`方法可能返回不符合 UTF-8 标准的字符串

具体来说，UTF-8 标准规定，`0xD800`到`0XDFFF`之间的码点，不能单独使用，必须配对使用。

为了确保返回的是合法的 UTF-8 字符串，ES2019 改变了`JSON.stringify`的行为。如果遇到`0xD800`到`0xDFFF`之间的单个码点，或者不存在的配对形式，它会返回转义字符串，留给应用自己决定下一步的处理。

```纯文本
JSON.stringify('\u{D834}') // "\uD834"
JSON.stringify('\uDF06\uD834') // "\uDF06\uD834"
```


模版字符串

模版字符串是增强版的字符串，用`标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```纯文本
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
not legal.`

// 嵌入变量
let name = 'Bob', time = 'today';
`Hello ${name}, how are you ${today}?`
```


> 


1. 如果使用模版字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中
2. 如果大括号里面的值不是字符串，将按照一般的规则转为字符串。（调用`toString`方法）

标签模版

标签模版其实不是模版，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模版字符串就是它的参数。

```纯文本
const tag = (strArr, ...arg) => console.log(strArr, arg);
let name = 'stoat';

tag(1); // [1]
tag`1`; // [["1"]]
tag`Hello ${name},how are you tody?`; // [["Hello ", ",how are you tody?"], "stoat"]
```


> 如果模版字符串里面有变量，就不是简单的调用了，而是会将模版字符串先处理成多个参数，再调用函数。


`tag`函数的第一个参数是一个数组，该数组的成员是模版字符串中那些没有变量替换的部分。

`tag`函数的其他参数，都是模版字符串各个变量被替换后的值。

“标签模版”的一个重要应用，就是过滤 HTML 字符串，防止用户输入恶意内容

```纯文本
const SaferHTML = (t, ...a) => {
	let s = t[0];

	for (let i = 0; i < a.length; i += 1){
		let arg = String(a[i]);

		s += arg.replace(/&/g, '&amp;')
						.replace(/</g, '&lt;')
						.replace(/>/g, '&gt;');

		s += t[i + 1]
	}
	return s;
}
let sender = '<script>alert("abc")</script>'; // 恶意代码
let msg = SaferHTML`<p>${sender} has sent you a message.</p>`;

msg; // "<p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>"
```


