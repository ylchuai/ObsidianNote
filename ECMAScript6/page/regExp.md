# regExp

正则的扩展

RegExp 构造函数

在 ES5 中，`RegExp`构造函数参数有两种形式。<br />第一种形式，参数是字符串，这个是第二个参数表示正则表达式的修饰符。

```纯文本
var regex = new RegExp('xyz', 'i');
// 等价于
var regex = /xyz/i;
```


第二种形式，参数是一个正则表达式，这是返回原有正则表达式的拷贝。

```纯文本
var regex = new RegExp(/xyz/i);
// 等价于
var regex = /xyz/i;
```


但是此时，ES5 不允许使用第二个参数，否则会报错

```纯文本
var regex = new RegExp(/xyz/, 'i')
// Uncaught TypeError: Cannot supply flags when constructing one RegExp from another
```


ES6改变了这种行为，第二个参数将会覆盖前面正则表达式的修饰符。

```纯文本
new RegExp(/abc/ig,'i').flags
// "i"
```


字符串的正则方法

字符串对象共有4个方法，可以使用正则表达式：`match()`、`replace()`、`search()`和`split()`。<br />ES6 将这4个方法，在语言内部全部调用`RegExp`的实例方法，从而做到所有于正则相关的方法，全都定义在`RegExp`对象上。

> String.prototype.match 调用 RegExp.prototype[Symbol.match]<br />String.prototype.replace 调用 RegExp.prototype[Symbol.replace]<br />String.prototype.search 调用 RegExp.prototype[Symbol.search]<br />String.prototype.split 调用 RegExp.prototype[Symbol.split]


u 修饰符

ES6 对正则表达式添加了`u`修饰符，含义为 Unicode 模式，用于处理大于`\uFFFF`的Unicode字符。

```纯文本
/^\uD83D/u.test('\uD83D\uDC2A') // false
/^\uD83D/.test('\uD83D\uDC2A') // true
```


RegExp.prototype.unicode 属性

正则实例对象新增`unicode`属性，表示是否设置了`u`修饰符

```纯文本
const r1 = /hello/;
const r2 = /hello/u;

r1.unicode; // false
r2.unicode; // true
```


y 修饰符

“粘连”修饰符。`y`修饰符与`g`修饰符作用类似，也是全局匹配，后一次匹配都是从上一次匹配成功的下一个位置开始。不同之处在于，`g`修饰符只要剩余位置存在匹配即可，而`y`修饰符确保匹配必须从剩余的第一个位置开始。

```纯文本
let s = 'aaa_aa_a';
let r1 = /a+/g;
let r2 = /a+/y;

r1.exec(s); // ["aaa"]
r2.exec(s); // ["aaa"]

r1.exec(s); // ["aa"]
r2.exec(s); // null
```


如果改一下正则表达式，保证每次都能头部匹配，`y`修饰符就会返回结果

```纯文本
let s = 'aaa_aa_a';
let r = /a+_/y;

r.exec(s); // ["aaa_"]
r.exec(s); // ["aa_"]
```


实际上，`y`修饰符隐含了头部匹配的标志`^`

```纯文本
/b/y.exec('aba');
// null
```


> 上面代码由于不能保证头部匹配，所以返回`null`。`y`修饰符设计的本意，就是让头部匹配的标志`^`在全局匹配中都有效。


下面是字符串对象的`replace`方法的例子

```纯文本
const REGEXP = /a/gy;

'aaxa'.replace(REGEXP, '-'); // "--xa"
```


单单一个`y`修饰符对`match`方法，只能返回第一个匹配，必须与`g`修饰符联用，才能返回所有匹配

```纯文本
'a1a2a3'.match(/a\d/y); // ["a1"]
'a1a2a3'.match(/a\d/gy); // ["a1", "a2", "a3"]
```


RegExp.prototype.sticky

是否设置了`y`修饰符

```纯文本
const REGEXP = /a/gy;

REGEXP.sticky; // true
```


RegExp.prototype.flags

返回正则表达式的修饰符

```纯文本
const REGEXP = /a/gy;

REGEXP.source; // "a"
REGEXP.flags; // "gy"
```


s 修饰符

正则表达式中，`.`代表任意单个字符。但有两个例外，一个是4个字节的 UTF-16 字符，这个可以用`u`修饰符解决。另外一个是行终止符。

> U+000A 换行符（\n）


U+000D 回车符（\r）<br />U+2028 行分隔符（line separator）<br />U+2029 段分隔符（paragraph separator）

```纯文本
/foo.bar/.test('foo\nbar');
// false
```


很多时候，我们希望匹配的是任意单个字符，ES2018 引入`s`修饰符，使得`.`可以匹配任意单个字符。

```纯文本
// /foo.bar/s.test('foo\nbar'); 
// true
```


RegExp.prototype.dotAll

正则实例对象新增`unicode`属性，表示是否设置了`s`修饰符

```纯文本
/foo.bar/s.dotAll;
// true
```


后行断言

JavaScript 语言的正则表达式，只支持先行断言和先行否定断言。ES2018 引入后行断言。

1. 先行断言：`x`只有在`y`前面才匹配，必须写成`/x(?=y)/`
2. 先行否定断言：`x`只有不在`y`前面才匹配，必须写成`/x(?!y)/`
3. 后行断言：`x`只有在`y`后面才匹配，必须写成`/(?<=y)x/`
4. 后行否定断言：`x`只有不在`y`后面才匹配，必须写成`/(?<!y)x/`<br />/\d+(?=%)/.exec('100% of US presidents have been male'); // ["100"]<br />/\d+(?!%)/.exec('that’s all 44 of them'); // ["44"]<br />/(?<=$)\d+/.exec('Benjamin Franklin is on the $100 bill'); // ["100"]<br /> /(?<!\$)\d+/.exec('it’s is worth about €90'); // ["90"]

Unicode 属性类

ES2018 引入了一种新的类的写法`\p{}`和`\P{}`，允许正则表达式匹配符合 Unicode 某种属性的所有字符

```纯文本
/\p{Script=Greek}/u.test('π'); // true
```


Unicode 属性类药指定属性名和属性值

```纯文本
/\p{UnicodePropertyName=UnicodePropertyValue}/
```


> 注意，这两种类，只对 Unicode 有效，所以使用的时候一定要加上`u`修饰符


```纯文本
/^\p{Decimal_Number}+$/u.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼'); // true
```


具名组匹配

正则表达式使用圆括号进行组匹配

```纯文本
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/

RE_DATE.exec('1999-12-31'); // ["1999-12-31", "1999", "12", "31"]
```


ES2018 引入了具名组匹配，允许为每个组匹配指定一个名字。

```纯文本
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const matchObj = RE_DATE.exec('1992-12-31');

matchObj.groups.year; // "year"
matchObj.groups.month; // "month"
matchObj.groups.day; // "day"
```


字符串替换时，使用`$<组名>`引用具名组

```纯文本
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;

'2015-01-02'.replace(re, '$<day>/$<month>/$<year>')
```


`replace`方法的第二个参数也可以是函数

```纯文本
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;

'2015-01-02'.replace(re, (
	matched, // 整个匹配结果 2015-01-02
	capture1, // 第一个组匹配 2015
	capture2, // 第二个组匹配	01
	capture3, // 第三个组匹配 02
	position, // 匹配开始的位置 0
	s, // 原字符串 2015-01-02
	groups // 具名组构成的一个对象 {year, month, day}
) => {
	let { day, month, year } = groups;

	return `${day}/${month}/${year}`;
})
```


如果要在正则表达式内部引用某个“具名组匹配”，可以使用`\k<组名>`的写法

```纯文本
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;

RE_TWICE.test('abc!abc'); // true
RE_TWICE.test('abc!ab'); // false
```


数字引用依然有效

```纯文本
const RE_TWICE = /^(?<word>[a-z]+)!\1$/;

RE_TWICE.test('abc!abc'); // true
RE_TWICE.test('abc!ab'); // false
```


这两种引用语法还可以同时使用

```纯文本
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;

RE_TWICE.test('abc!abc!abc'); // true
RE_TWICE.test('abc!bac!ab'); // false
```


